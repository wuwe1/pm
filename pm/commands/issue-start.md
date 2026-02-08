---
allowed-tools: Bash, Read, Write, LS, Task
---

# Issue Start

开始处理单个 GitHub Issue。

## 用法

```
/pm:issue-start <issue_number>              # 当前 session 直接工作
/pm:issue-start <issue_number> --team       # 启动 mini Agent Team
```

## 前置检查

1. **获取 Issue 详情**：
   ```bash
   gh issue view $ARGUMENTS --json state,title,labels,body
   ```
   失败时："✗ 无法访问 Issue #$ARGUMENTS。检查编号或运行：gh auth login"

2. **查找本地任务文件**：
   - 查找 `.claude/epics/*/$ARGUMENTS.md`（Issue 编号命名）
   - 未找到时："✗ 未找到 Issue #$ARGUMENTS 的本地任务文件"

## 默认模式（当前 session 直接工作）

### 1. 创建分支

```bash
# 从 epic 分支创建（如有），否则从 main 创建
epic_name={从任务文件路径提取}

if git branch -a | grep -q "epic/$epic_name"; then
  git checkout epic/$epic_name
  git pull origin epic/$epic_name
  git checkout -b issue/$ARGUMENTS
else
  git checkout main
  git pull origin main
  git checkout -b issue/$ARGUMENTS
fi
git push -u origin issue/$ARGUMENTS
```

### 2. 读取任务

- 读取任务文件内容
- 理解需求和验收标准
- 识别依赖和涉及的文件

### 3. 遵循 6.031 原则工作

按照 `rules/6031-principles.md`：

1. **规范**：明确要实现的接口契约（前置/后置条件）
2. **测试**：先写测试用例（等价类划分 + 边界值）
3. **实现**：编码通过测试
4. **验证**：checkRep（如涉及 ADT）

### 4. Commit 与更新

- Commit 格式：`Issue #$ARGUMENTS: {具体更改}`
- 完成后更新任务 frontmatter：`status` → `done`
- 更新 Epic frontmatter：`tasks_done` += 1
- 更新 GitHub：
  ```bash
  gh issue edit $ARGUMENTS --add-label "done"
  ```

### 5. 输出

```
✓ Issue #$ARGUMENTS 已完成

分支：issue/$ARGUMENTS
提交：{commit_count} 个 commit
变更文件：{file_count} 个

下一步：
  - 创建 PR 合并到 epic 分支
  - 或继续其他 issue：/pm:issue-start {next_issue}
```

## --team 模式

### 1. 检查 Agent Teams

检测 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` 环境变量或 settings.json 配置。

未启用时输出：
```
✗ Agent Teams 未启用。请在 settings.json 中添加：
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
或设置环境变量：export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
然后重启 Claude Code。
```

### 2. 创建分支

同默认模式。

### 3. 分析任务

- 读取 task 文件内容
- 识别可并行的工作流（如：前端 + 后端 + 测试）

### 4. 创建 Mini Agent Team

为每个工作流创建一个 teammate：
- 每个 teammate 负责一个明确的工作范围
- 预加载 `skills/6031-review.md`
- Commit 格式：`Issue #$ARGUMENTS: {具体更改}`
- 要求 plan approval 再开始

### 5. Team Lead 协调

- 监控 teammate 进度
- 处理冲突和依赖
- 所有工作流完成后更新 frontmatter

## 错误处理

失败时报告："✗ {失败原因}：{解决方案}"
