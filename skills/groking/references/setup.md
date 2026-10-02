# 配置 Grok CLI

当 `scripts/groking.sh` 以 `not_installed`（3）、`not_signed_in`（4）或 `x_search_unavailable`（6）退出时读本文件。它说明如何把用户从"什么都没装"带到"查询可用"，以及登录正常却查不到 X 时怎么办。

## 引导原则

- **安装和登录由用户自己做，不是你做。** 安装是从网上下载脚本在用户机器上运行，登录要在用户的浏览器里用本人账号完成。给出命令、用一句话说明它做什么，然后等用户告诉你做完了。只有用户明确要求时才由你运行安装命令。
- **不要在对话里索要密码、token 或 API key。** 用户主动给出时，请他改放到 shell 环境变量里。
- **一步一验证。** 每步之后运行 `scripts/groking.sh --check` 确认，再进行下一步。
- **用用户的语言，话说短。** 大多数人只需要两条命令。
- **出问题时转述事实，不转述猜测。** 脚本的 `[groking]` 状态行和日志路径原样给用户；原因不确定就说不确定。

先告诉用户现在的情况，例如："读 X 需要 Grok CLI，这台机器上还没有装。两步：装上它，然后登录你的 Grok 账号。"

## 安装

请用户在自己的终端里运行对应平台的命令。

macOS、Linux、WSL，或 Windows 上的 Git Bash：

```bash
curl -fsSL https://x.ai/cli/install.sh | bash
```

Windows PowerShell：

```powershell
irm https://x.ai/cli/install.ps1 | iex
```

这是 Grok CLI（`grok` 命令，也叫 Grok Build）的官方安装脚本，由 x.ai 提供。它把 CLI 装在 `~/.grok/` 下，可执行文件在 `~/.grok/bin/grok`。

用户说装好后，运行 `scripts/groking.sh --check`。

- 脚本除了 PATH 还会查 `~/.grok/bin`，所以即使你的 shell 是安装前启动的，也能找到刚装的 CLI。
- 如果用户自己的终端提示 `grok: command not found`，请他新开一个终端窗口。

## 登录

请用户在自己的终端里运行：

```bash
grok login
```

它会打开浏览器页面，登录 Grok 账号。凭据保存在用户机器的 `~/.grok/auth.json`，你不会接触到。

机器上没有浏览器时（SSH 会话、容器、远程虚拟机）：

```bash
grok login --device-auth
```

它会打印一个网址和一个验证码，用户在任意设备上打开网址、输入验证码即可。

需要知道的几点：

- 登录是交互式的，你在后台运行的命令完成不了，必须在用户的终端里进行。
- Grok CLI 也接受 `XAI_API_KEY` 环境变量里的 API key。本 skill 只用账号登录测试过，API key 方式下 X 搜索是否可用没有验证，所以建议账号登录。
- Grok CLI 的使用资格和额度取决于用户的 Grok 套餐。如果登录成功但查询报了套餐或额度相关的错误，把错误原样转述给用户。

## 验证

```bash
scripts/groking.sh --check
```

预期输出：

```
grok: grok <版本>
auth: signed in
ready
```

这一步不消耗额度，但也只能确认"装了、登录了"。要确认 X 搜索真的可用，再运行一次 `scripts/groking.sh --probe`（消耗一次最小查询）。首次为用户配置时建议跑一次，省得等到真正的问题上才发现查不了。

然后执行用户最初要的那次查询。

## X 搜索不可用

现象：脚本退出码 6、状态 `x_search_unavailable`；或者 Grok 的回答里说"没有可用的 X 搜索工具"之类的话。此时登录是正常的，问题出在这个账号在 Grok CLI 里拿不到 X 搜索这个服务端工具。

处理步骤：

1. 运行 `scripts/groking.sh --probe` 确认。退出码 6 就是确认了；退出码 0 说明刚才是偶发，直接重试原来的问题。
2. 确认后，把事实告诉用户：Grok CLI 已经登录，但这个账号在 CLI 里没有 X 搜索能力；不是 skill 的问题，也不是本机配置的问题。
3. 请用户自己验证一次：在终端运行 `grok` 进入交互界面，直接问"@X 账号最新一条推文是什么"。
   - 交互界面里也查不到：是账号能力问题。已知有用户反馈，没有 X Premium+ 会员的账号登录后出现过这种情况（未经官方确认）。请用户在 grok.com 查看自己的套餐。
   - 交互界面里能查到：是无头模式或本 skill 的问题。请用户到仓库提 issue，附上脚本给出的日志文件。
4. 不要反复重试。每次重试都消耗用户的额度，而结论不会变。

## 排查

| 状态 | 可能原因 | 处理 |
| --- | --- | --- |
| 刚装完仍是 `not_installed` | 安装失败，或装到了别的位置 | 请用户贴出安装命令的输出，检查 `~/.grok/bin/grok` 是否存在 |
| 登录后仍是 `not_signed_in` | 浏览器流程没走完，或会话已过期 | 请用户先运行 `grok logout`，再运行 `grok login` |
| `outdated` | 已安装版本缺少脚本用到的参数 | 请用户运行 `grok update` |
| `x_search_unavailable` | 账号在 Grok CLI 里没有 X 搜索能力 | 见上一节，不要重试 |
| `warn: 本次没有调用 X 搜索` | Grok 没搜 X 就作答了 | 结果可能不是来自 X。把 warn 行告诉用户；若反复出现，运行 `--probe` |
| `sandbox_unavailable` | 这个系统上 Grok 的沙箱起不来 | 在环境变量里加 `GROKING_SANDBOX=off` 重跑一次，并告诉用户你这么做了 |
| `incomplete` 或 `empty` | 问题太宽，或运行被中断 | 缩小问题范围重试一次 |
| `failed` 且信息是网络或服务端错误 | 网络或 xAI 侧的临时问题 | 稍等后重试一次，再失败就转述错误信息 |

每次失败脚本都会保留日志（路径在状态行末尾，默认在 `$TMPDIR/groking/`），里面有 Grok 版本、问题、完整的事件流和 stderr。用户提 issue 时附上它。设置 `GROKING_DEBUG=1` 可以让成功的查询也保留日志。
