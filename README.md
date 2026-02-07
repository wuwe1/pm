# PM

Claude Code 项目管理系统。规范驱动，Agent Teams 并行，集成 MIT 6.031 原则。

## 工作流程

```
prd-new → prd-parse → epic-decompose → epic-sync → epic-start → epic-merge
```

| 命令 | 说明 |
|------|------|
| `/pm:prd-new` | 头脑风暴创建 PRD |
| `/pm:prd-parse` | PRD → 技术 Epic |
| `/pm:epic-decompose` | Epic → 任务文件 |
| `/pm:epic-sync` | 推送到 GitHub Issues |
| `/pm:epic-start` | 启动 Agent Team |
| `/pm:issue-start` | 处理单个 Issue（`--team` 可选） |
| `/pm:epic-merge` | 合并到 main |

## 安装

```bash
git clone https://github.com/wuwe1/pm.git /tmp/pm-install
cp -r /tmp/pm-install/pm/ /path/to/your/project/.claude/
rm -rf /tmp/pm-install
```

需要 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 和 [GitHub CLI](https://cli.github.com/)。

Agent Teams 启用：`export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

## 许可证

MIT
