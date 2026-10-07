# groking

让你的编程 agent 读得到 X (Twitter)。

Claude Code、Codex 这类 agent 读不到 X 上的推文，网页工具要么被拦截，要么只拿到空页面。Grok 原生支持搜索 X。这个 skill 让你的 agent 把 X 上的查询交给本机的 Grok CLI，拿到结果后继续工作。

## 能做什么

- 按链接读一条推文或整个 thread、看某个账号的近期动态、看某个话题的讨论、看一条推文下的回复。
- 几个问题一次传入，并发查询。
- 没装 Grok CLI 或没登录时，agent 会一步步带你完成，然后回到你的问题。
- Grok 运行时本地工具全部被移除，它只能搜索，做不了别的。

## 需要什么

- 一个支持 [Agent Skills](https://agentskills.io) 的 agent。在 macOS 上的 Claude Code 和 Codex 中测试过，其他 agent 和平台理论上可用，但没有测试过。
- Grok CLI 和一个 Grok 账号。不需要提前准备，skill 会引导你。
- Grok CLI 里的 X 搜索能力取决于账号套餐。有用户反馈，没有 X Premium+ 会员的账号登录后查不到 X（未经官方确认）。这时 skill 会明确告诉你，而不是给出错误的结果。

## 安装

用 [skills CLI](https://github.com/vercel-labs/skills)，安装时选择你用的 agent：

```bash
npx skills add wolfyxbt/groking -g
```

或者手动复制：

```bash
git clone https://github.com/wolfyxbt/groking.git
cp -r groking/skills/groking ~/.claude/skills/   # Claude Code
cp -r groking/skills/groking ~/.codex/skills/    # Codex
```

### Codex 用户

Codex 默认的沙箱禁止联网，也不让 Grok 写自己的目录，所以脚本第一次运行会失败并提示"请在沙箱外运行"，Codex 会向你申请批准，同意即可。不想每次批准，在 `~/.codex/config.toml` 里加上：

```toml
[sandbox_workspace_write]
network_access = true
writable_roots = ["~/.grok"]
```

这样脚本就能在沙箱内运行。

## 使用

直接用自然语言问你的 agent：

- "这条推文说了什么？https://x.com/..."
- "@AnthropicAI 这周发了什么？"
- "今天 X 上大家怎么讨论 Claude Code？"

简单查询 15–60 秒，需要多轮搜索的开放问题可能要几分钟。

## 安全与隐私

推文是不可信的文本，读了它的 agent 有可能被内容带偏，所以 Grok 在没有任何行动能力的状态下运行：

- 移除所有客户端工具：shell、文件读写、网页抓取、子 agent、MCP 工具。X 搜索运行在 xAI 的服务器上，不受影响。
- 每次调用显式指定权限模式，你 Grok 配置里的默认放行不会生效。
- 在空的临时目录里运行，关闭跨会话记忆，接触不到你的项目文件。
- agent 只调用 skill 自带的脚本，不直接调用 `grok`，并被要求把返回内容当作信息而不是指令。

你的问题通过你自己的 Grok 账号发送给 xAI，消耗你套餐的额度（用 API key 登录则按 token 计费）。Grok CLI 的安装和登录都由你自己完成，skill 接触不到你的凭据。

本项目与 xAI 和 X 没有关联。

## 许可证

[MIT](LICENSE)
