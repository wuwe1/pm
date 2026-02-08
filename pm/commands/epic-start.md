---
allowed-tools: Bash, Read, Write, LS, Task
---

# Epic Start

启动 Agent Team 并行处理 Epic 的所有任务。

## 用法

```
/pm:epic-start <epic_name>
```

## 前置检查

1. **验证 Epic 存在**：
   ```bash
   test -f .claude/epics/$ARGUMENTS/epic.md || echo "✗ Epic 未找到。运行：/pm:prd-parse $ARGUMENTS"
   ```

2. **验证已同步到 GitHub**：检查 Epic frontmatter 中 `github` 字段非空
   - 为空时："✗ Epic 未同步。先运行：/pm:epic-sync $ARGUMENTS"

3. **检查未提交更改**：
   ```bash
   [ -n "$(git status --porcelain)" ] && echo "✗ 有未提交的更改。请先 commit 或 stash。"
   ```

4. **检查 Agent Teams 是否启用**：

   检测方式：检查 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` 环境变量或 settings.json 中的配置。

   如未启用，输出：
   ```
   ✗ Agent Teams 未启用。请在 settings.json 中添加：
   { "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
   或设置环境变量：export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
   然后重启 Claude Code。
   ```

## 指令

### 1. 创建 Epic 分支

```bash
git checkout main && git pull origin main
git checkout -b epic/$ARGUMENTS
git push -u origin epic/$ARGUMENTS
echo "✓ 已创建分支：epic/$ARGUMENTS"
```

如分支已存在：
```bash
git checkout epic/$ARGUMENTS
git pull origin epic/$ARGUMENTS
```

### 2. 分析任务依赖图

读取 `.claude/epics/$ARGUMENTS/` 下所有任务文件：
- 解析每个任务的 `depends_on` 和 `parallel` 字段
- 分类：
  - **Ready**：无未完成依赖，可立即开始
  - **Blocked**：有未完成的前置依赖

### 3. 创建 Agent Team

用自然语言指示创建 Agent Team。为每个 Ready 任务生成一个 teammate：

每个 teammate 收到以下信息：
- 完整的 task 文件内容
- Epic 上下文（epic.md 摘要）
- 工作分支：`epic/$ARGUMENTS`
- Commit 格式：`Issue #{number}: {具体更改}`
- 预加载 `skills/6031-review.md` 以便自审代码
- **要求 plan approval 再开始实现**

示例 teammate 指令：

```
你负责处理 Issue #{number}：{任务标题}

工作分支：epic/$ARGUMENTS

任务内容：
{task 文件的完整内容}

要求：
1. 先制定实现计划，获得 approval 后再编码
2. 遵循 6.031 原则（规范 → 测试 → 实现）
3. Commit 格式：Issue #{number}: {具体更改}
4. 完成后更新任务文件 frontmatter：status → done
5. 完成后更新 epic frontmatter：tasks_done += 1
```

### 4. 依赖管理

Team Lead（主 session）监控任务完成情况：

- Teammate 完成任务后：
  1. 更新 task frontmatter `status` → `done`
  2. 更新 epic frontmatter `tasks_done` += 1
  3. 检查是否有新的 unblocked 任务
  4. 为新 unblocked 任务分配新 teammate

### 5. 完成通知

所有任务完成后输出：

```
✓ Epic $ARGUMENTS 所有任务已完成！

完成情况：
  - 总任务：{total}
  - 已完成：{done}

下一步：运行 /pm:epic-merge $ARGUMENTS 合并到 main
```

## 错误处理

- Agent 启动失败时，报告具体错误并继续其他任务
- 任务执行失败时，保留进度，报告失败原因
- 不自动重试，由 Team Lead 决定下一步
