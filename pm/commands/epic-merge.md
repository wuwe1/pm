---
allowed-tools: Bash, Read, Write
---

# Epic Merge

合并已完成的 Epic 到 main 分支。

## 用法

```
/pm:epic-merge <epic_name>
```

## 前置检查

1. **验证 Epic 分支存在**：
   ```bash
   git branch -a | grep "epic/$ARGUMENTS" || echo "❌ 未找到 epic/$ARGUMENTS 分支"
   ```

2. **提醒清理 Agent Team**：如果之前使用了 Agent Team，确保所有 teammate 已完成工作。

## 指令

### 1. 合并前验证

```bash
# 切换到 epic 分支检查状态
git checkout epic/$ARGUMENTS

# 检查未提交更改
if [ -n "$(git status --porcelain)" ]; then
  echo "⚠️ Epic 分支有未提交更改："
  git status --short
  echo "请先 commit 或 stash"
  exit 1
fi

# 同步远程
git fetch origin
```

### 2. 运行测试（推荐）

根据项目类型自动检测并运行测试：

```bash
if [ -f package.json ]; then
  npm test
elif [ -f pyproject.toml ]; then
  uv run pytest
elif [ -f Cargo.toml ]; then
  cargo test
elif [ -f go.mod ]; then
  go test ./...
fi
```

测试失败时询问是否继续。

### 3. 更新 Epic 文档

获取当前时间：`date -u +"%Y-%m-%dT%H:%M:%SZ"`

更新 `.claude/epics/$ARGUMENTS/epic.md` frontmatter：
- `status` → `done`

### 4. 执行合并

```bash
# 回到 main 分支
git checkout main
git pull origin main

# 合并（保留历史）
git merge epic/$ARGUMENTS --no-ff -m "Merge epic: $ARGUMENTS"
```

### 5. 处理合并冲突

如果合并失败：

```
❌ 合并冲突！

冲突文件：
{冲突文件列表}

选项：
1. 手动解决：编辑冲突文件 → git add → git commit
2. 中止合并：git merge --abort
```

### 6. 合并后清理

合并成功后：

```bash
# 推送到远程
git push origin main

# 删除分支
git branch -d epic/$ARGUMENTS
git push origin --delete epic/$ARGUMENTS 2>/dev/null || true

# 归档 Epic
mkdir -p .claude/epics/archived/
mv .claude/epics/$ARGUMENTS .claude/epics/archived/
```

### 7. 关闭 GitHub Issues

```bash
# 关闭 Epic Issue
epic_github=$(grep '^github:' .claude/epics/archived/$ARGUMENTS/epic.md | grep -oE '[0-9]+$' || true)
[ -n "$epic_github" ] && gh issue close "$epic_github" -c "Epic 已完成并合并到 main"

# 关闭任务 Issues
for task_file in .claude/epics/archived/$ARGUMENTS/[0-9]*.md; do
  [ -f "$task_file" ] || continue
  issue_num=$(grep '^github:' "$task_file" | grep -oE '[0-9]+$' || true)
  [ -n "$issue_num" ] && gh issue close "$issue_num" -c "Epic 合并完成"
done
```

### 8. 最终输出

```
✅ Epic 合并成功：$ARGUMENTS

汇总：
  分支：epic/$ARGUMENTS → main
  合并提交：{count}
  变更文件：{count}
  关闭 Issues：{count}

清理完成：
  ✓ 分支已删除
  ✓ Epic 已归档
  ✓ GitHub Issues 已关闭

下一步：
  - 部署（如需要）
  - 开始新功能：/pm:prd-new {feature}
```

## 重要事项

- 使用 `--no-ff` 保留 Epic 历史
- 归档 Epic 数据而非删除
- 关闭 GitHub Issues 保持同步
