# PM

Claude Code 项目管理系统。

## 命令

| 命令 | 说明 |
|------|------|
| `/pm:prd-new` | 创建 PRD |
| `/pm:prd-parse` | PRD → Epic |
| `/pm:epic-decompose` | Epic → 任务 |
| `/pm:epic-sync` | 推送到 GitHub |
| `/pm:epic-start` | 启动 Agent Team |
| `/pm:issue-start` | 处理单个 Issue |
| `/pm:epic-merge` | 合并到 main |

## 安装

```bash
git clone https://github.com/wuwe1/pm.git /tmp/pm
cp -r /tmp/pm/pm/ your-project/.claude/
rm -rf /tmp/pm
```
