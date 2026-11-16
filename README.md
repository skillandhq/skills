# Skilland skills

Free skills made by [Skilland](https://skilland.app), the catalog of AI agent skills. MIT licensed.

Skilland 自己做的免费 skill，MIT 许可证。

| Skill | What it does |
|---|---|
| [`skilland-finder`](skills/skilland-finder/SKILL.md) | Lets your agent search Skilland for a skill, show its capability labels and review status, and install it once you agree.<br>让 Agent 在 Skilland 里搜索 skill，列出能力标签和审核情况，经你同意后装好。 |

## Install

- **Skills CLI** (Node.js 22.20 or later): run `npx skills add skillandhq/skills` and pick `skilland-finder`.
- **By hand:** copy `skills/skilland-finder/` into your agent's skills folder so that `SKILL.md` sits directly inside it, for example `~/.claude/skills/skilland-finder/` for Claude Code, or `~/.agents/skills/skilland-finder/` for agents that read the shared folder. Then start a new session.

## 安装

- **skills 命令行**（Node.js 22.20 或更高）：运行 `npx skills add skillandhq/skills`，选 `skilland-finder`。
- **手动：** 把 `skills/skilland-finder/` 复制到 Agent 的 skill 目录，让 `SKILL.md` 直接在这个文件夹里，比如 Claude Code 的 `~/.claude/skills/skilland-finder/`，或者读通用目录的 Agent 用 `~/.agents/skills/skilland-finder/`。然后新开一个会话。

## Examples / 示例

- "Find me a skill for making slide decks"
- "Install the pptx skill from Skilland into Codex"
- 「帮我找一个能做 PPT 的 skill」
- 「Skilland 上有没有写小红书笔记的 skill？」

## License

MIT, see [LICENSE](LICENSE).
