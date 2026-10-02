# 配置 Grok CLI

当脚本以 3（未安装）、4（未登录）或 6（没有 X 搜索能力）退出时读本文件。

## 引导原则

- **安装和登录由用户自己做。** 安装是从网上下载脚本在用户机器上运行，登录要在用户的浏览器里用本人账号完成。给出命令，一句话说明它做什么，等用户说做完了。只有用户明确要求时才由你运行安装命令。
- **不要在对话里索要密码、token 或 API key。**
- **一步一验证**，每步之后运行 `scripts/groking.sh --check`。

开场可以这样说："读 X 需要 Grok CLI，这台机器上还没有装。两步：装上它，然后登录你的 Grok 账号。"

## 安装

macOS、Linux、WSL，或 Windows 上的 Git Bash：

```bash
curl -fsSL https://x.ai/cli/install.sh | bash
```

Windows PowerShell：

```powershell
irm https://x.ai/cli/install.ps1 | iex
```

这是 Grok CLI（`grok` 命令，也叫 Grok Build）的官方安装脚本，由 x.ai 提供，装在 `~/.grok/`。脚本除了 PATH 还会查 `~/.grok/bin`，所以刚装完就能用；用户自己的终端如果提示 `command not found`，新开一个窗口即可。

## 登录

```bash
grok login
```

打开浏览器登录 Grok 账号，凭据存在用户机器的 `~/.grok/auth.json`。没有浏览器的环境（SSH、容器）用 `grok login --device-auth`，按提示在别的设备上输入验证码。

登录是交互式的，必须在用户的终端里完成。Grok CLI 也接受 `XAI_API_KEY` 环境变量，但本 skill 只用账号登录测试过。

## 验证

```bash
scripts/groking.sh --check
```

输出 `ready` 后，执行用户最初要的那次查询。

## 没有 X 搜索能力（退出码 6）

登录正常，但这个账号在 Grok CLI 里拿不到 X 搜索。已知没有 X Premium+ 会员的账号会这样（未经官方确认）。如实告诉用户，请他到 grok.com 确认套餐；不要反复重试，每次都消耗额度而结论不变。

如果用户确认套餐没问题，可以请他在终端运行 `grok` 进入交互界面，直接问一个 X 上的问题：交互界面里能查到，就是本 skill 的问题，请他到仓库提 issue。

## 其他情况

| 现象 | 处理 |
| --- | --- |
| 刚装完仍提示未安装 | 请用户贴出安装命令的输出，检查 `~/.grok/bin/grok` 是否存在 |
| 登录后仍提示未登录 | 请用户先 `grok logout`，再 `grok login` |
| 提示"这次回答没有经过 X 搜索" | 结果可能不是来自 X，转述时说明；反复出现则按退出码 6 处理 |
| 网络或服务端错误 | 稍等后重试一次 |
