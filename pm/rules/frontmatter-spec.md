# 文件约定规范

本规则定义 PM 系统中所有 Markdown 文件的 Frontmatter 规范和文件路径命名规范。

## 时间戳格式

所有时间字段使用 ISO 8601 UTC 格式：

```bash
date -u +"%Y-%m-%dT%H:%M:%SZ"
```

**严禁**使用占位符文本，必须获取系统真实时间。

## PRD Frontmatter

文件路径：`.claude/prds/{feature-name}.md`

```yaml
---
type: prd                              # 固定值，标识文件类型
name: feature-name                     # kebab-case，同文件名
description: "一行中文简述"              # 双引号包裹
status: backlog | in-progress | done   # 三态枚举
created: 2026-02-07T12:00:00Z         # ISO 8601 UTC
---
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `type` | string | 是 | 固定 `prd` |
| `name` | string | 是 | kebab-case，同文件名 |
| `description` | string | 是 | 双引号包裹的中文一行简述 |
| `status` | enum | 是 | `backlog` / `in-progress` / `done` |
| `created` | datetime | 是 | ISO 8601 UTC |

## Epic Frontmatter

文件路径：`.claude/epics/{feature-name}/epic.md`

```yaml
---
type: epic                             # 固定值
name: feature-name                     # 同 PRD name
status: backlog | in-progress | done   # 三态枚举
created: 2026-02-07T12:00:00Z         # ISO 8601 UTC
prd: .claude/prds/feature-name.md     # PRD 相对路径
github: ""                             # sync 后填充 Issue URL
task_count: 0                          # 任务总数（decompose 后更新）
tasks_done: 0                          # 已完成任务数
---
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `type` | string | 是 | 固定 `epic` |
| `name` | string | 是 | 同 PRD name |
| `status` | enum | 是 | `backlog` / `in-progress` / `done` |
| `created` | datetime | 是 | ISO 8601 UTC |
| `prd` | string | 是 | PRD 文件相对路径 |
| `github` | string | 是 | sync 后填充 GitHub Issue URL，初始为空 |
| `task_count` | integer | 是 | 任务总数，decompose 后更新 |
| `tasks_done` | integer | 是 | 已完成任务数 |

## Task Frontmatter

文件路径：`.claude/epics/{feature-name}/{number}.md`
- 初始为顺序编号（`001.md`, `002.md`）
- sync 后重命名为 GitHub Issue 编号（`{issue-number}.md`）

```yaml
---
type: task                             # 固定值
name: "任务标题中文"                     # 双引号包裹
status: open | in-progress | done      # 三态枚举
created: 2026-02-07T12:00:00Z         # ISO 8601 UTC
github: ""                             # sync 后填充 Issue URL
depends_on: []                         # 依赖的任务编号列表
parallel: true                         # 是否可并行
---
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `type` | string | 是 | 固定 `task` |
| `name` | string | 是 | 双引号包裹的任务标题 |
| `status` | enum | 是 | `open` / `in-progress` / `done` |
| `created` | datetime | 是 | ISO 8601 UTC |
| `github` | string | 是 | sync 后填充 GitHub Issue URL，初始为空 |
| `depends_on` | array | 是 | 依赖的任务编号列表 |
| `parallel` | boolean | 是 | 是否可与其他任务并行 |

## 文件路径命名规范

| 文件类型 | 路径模式 | 命名规则 |
|----------|----------|----------|
| PRD | `.claude/prds/{name}.md` | kebab-case |
| Epic | `.claude/epics/{name}/epic.md` | 同 PRD name |
| Task | `.claude/epics/{name}/{number}.md` | 顺序编号或 Issue 编号 |
| GitHub Mapping | `.claude/epics/{name}/github-mapping.md` | 固定名称 |

## 状态流转

```
PRD:   backlog → in-progress → done
Epic:  backlog → in-progress → done
Task:  open    → in-progress → done
```

## Frontmatter 剥离

创建 GitHub Issue 时，需要剥离 Frontmatter：

```bash
sed '1,/^---$/d; 1,/^---$/d' file.md
```

## FSEvent 监控说明

外部工具可通过 FSEvent 监听以下路径，实时追踪项目进度：

- **监听路径**：`.claude/prds/` 和 `.claude/epics/` 的文件创建/修改事件
- **解析策略**：检测到文件变更时，读取 frontmatter 的 `type` 字段区分文件类型，然后读取 `status` 字段
- **进度计算**：Epic 的 `task_count` 和 `tasks_done` 字段可直接算百分比

### 外部查询命令示例

```bash
# 查看所有 Epic 状态
for f in .claude/epics/*/epic.md; do
  name=$(grep '^name:' "$f" | sed 's/^name: *//')
  status=$(grep '^status:' "$f" | sed 's/^status: *//')
  done=$(grep '^tasks_done:' "$f" | sed 's/^tasks_done: *//')
  total=$(grep '^task_count:' "$f" | sed 's/^task_count: *//')
  echo "$name: $status ($done/$total)"
done

# 查看所有 PRD 状态
for f in .claude/prds/*.md; do
  name=$(grep '^name:' "$f" | sed 's/^name: *//')
  status=$(grep '^status:' "$f" | sed 's/^status: *//')
  echo "$name: $status"
done
```
