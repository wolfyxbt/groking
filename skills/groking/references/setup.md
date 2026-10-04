# 配置 Grok CLI

脚本以 3（未安装）或 4（未登录）退出时读本文件。

两条原则：**安装和登录由用户自己做**，你给命令、一句话说明、等他做完；**不要在对话里索要密码、token 或 API key**。

## 安装

macOS、Linux、WSL，或 Windows 上的 Git Bash：

```bash
curl -fsSL https://x.ai/cli/install.sh | bash
```

Windows PowerShell：

```powershell
irm https://x.ai/cli/install.ps1 | iex
```

这是 Grok CLI（`grok` 命令，也叫 Grok Build）的官方安装脚本，装在 `~/.grok/`。脚本除了 PATH 还会查 `~/.grok/bin`，所以装完立刻能用；用户自己的终端若提示 `command not found`，新开一个窗口即可。

## 登录

```bash
grok login
```

打开浏览器登录 Grok 账号，凭据存在用户机器的 `~/.grok/auth.json`。没有浏览器的环境（SSH、容器）用 `grok login --device-auth`，按提示在别的设备上输入验证码。

## 验证

重新运行用户刚才的问题。成功就继续；仍提示未登录，请用户先 `grok logout` 再 `grok login`。
