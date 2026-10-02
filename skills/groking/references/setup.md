# 配置 Grok CLI

当 `scripts/groking.sh` 以 `not_installed`（3）或 `not_signed_in`（4）退出时读本文件。它说明如何把用户从"什么都没装"带到"查询可用"。

## 引导原则

- **安装和登录由用户自己做，不是你做。** 安装是从网上下载脚本在用户机器上运行，登录要在用户的浏览器里用本人账号完成。给出命令、用一句话说明它做什么，然后等用户告诉你做完了。只有用户明确要求时才由你运行安装命令。
- **不要在对话里索要密码、token 或 API key。** 用户主动给出时，请他改放到 shell 环境变量里。
- **一步一验证。** 每步之后运行 `scripts/groking.sh --check` 确认，再进行下一步。
- **用用户的语言，话说短。** 大多数人只需要两条命令。

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

然后执行用户最初要的那次查询。

## 排查

| 状态 | 可能原因 | 处理 |
| --- | --- | --- |
| 刚装完仍是 `not_installed` | 安装失败，或装到了别的位置 | 请用户贴出安装命令的输出，检查 `~/.grok/bin/grok` 是否存在 |
| 登录后仍是 `not_signed_in` | 浏览器流程没走完，或会话已过期 | 请用户先运行 `grok logout`，再运行 `grok login` |
| `outdated` | 已安装版本缺少脚本用到的参数 | 请用户运行 `grok update` |
| `sandbox_unavailable` | 这个系统上 Grok 的沙箱起不来 | 在环境变量里加 `GROKING_SANDBOX=off` 重跑一次，并告诉用户你这么做了 |
| `incomplete` 或 `empty` | 问题太宽，或运行被中断 | 缩小问题范围重试一次 |
| `failed` 且信息是网络或服务端错误 | 网络或 xAI 侧的临时问题 | 稍等后重试一次，再失败就转述错误信息 |
