# video-factory-skills

**Video Factory AIGC skills** — automatic video editing, plan-driven rough/final cuts, quality judgement, recovery, and verified media receipts.

本包包含 **5 个技能**（video-shots/video-sync 在第三方 reelbench-skills 中）。

## 📦 安装

```bash
npx skills add full-aigc-skills/video-factory-skills
```

## 🎯 技能列表 (5)

| 技能 | 描述 |
|------|------|
| `video-factory-use` | 路由器：分发视频剪辑请求到最窄适用 Skill |
| `video-factory-plan` | 把本地素材转成已验证 rough-cut 或 final-cut 编辑决策 |
| `video-factory-run` | 按已批准 EditDecision 渲染、收集、记录（final cut 需要 approval 文件） |
| `video-factory-judge` | 媒体门控 + 语义评审 + 人工接收 |
| `video-factory-recover` | 中断后的合法下一步判定（durable ledger） |

## 🤖 支持的智能体

Claude Code / Codex / Cursor / OpenCode / Gemini CLI / GitHub Copilot / Windsurf。

## 📄 License

Apache 2.0
