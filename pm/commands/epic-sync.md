---
allowed-tools: Bash, Read, Write, LS, Task
---

# Epic Sync

推送 Epic 和任务到 GitHub 作为 Issues。

## 用法

```
/pm:epic-sync <feature_name>
```

## 前置检查

```bash
# 验证 Epic 存在
test -f .claude/epics/$ARGUMENTS/epic.md || echo "✗ Epic 未找到。运行：/pm:prd-parse $ARGUMENTS"

# 统计任务文件
ls .claude/epics/$ARGUMENTS/[0-9]*.md 2>/dev/null | wc -l
```

无任务时："✗ 无任务可同步。运行：/pm:epic-decompose $ARGUMENTS"

## 指令

### 0. 仓库保护检查

遵循 `rules/github-operations.md`，确保不会同步到 PM 模板仓库：

```bash
remote_url=$(git remote get-url origin 2>/dev/null || echo "")
if [[ "$remote_url" == *"wuwe1/pm"* ]]; then
  echo "✗ 错误：不能同步到模板仓库！"
  echo "请更新 remote origin 到你自己的仓库。"
  exit 1
fi
```

### 1. 创建 Epic Issue

```bash
# 检测 GitHub 仓库
remote_url=$(git remote get-url origin 2>/dev/null || echo "")
REPO=$(echo "$remote_url" | sed 's|.*github.com[:/]||' | sed 's|\.git$||')
[ -z "$REPO" ] && echo "✗ 无法检测 GitHub 仓库" && exit 1

# 剥离 frontmatter，准备 Issue body
sed '1,/^---$/d; 1,/^---$/d' .claude/epics/$ARGUMENTS/epic.md > /tmp/epic-body.md

# 创建 Epic Issue
epic_number=$(gh issue create \
  --repo "$REPO" \
  --title "Epic: $ARGUMENTS" \
  --body-file /tmp/epic-body.md \
  --label "epic,epic:$ARGUMENTS" \
  --json number -q .number)
```

### 2. 创建任务 Sub-Issues

检查 gh-sub-issue 扩展：
```bash
if gh extension list | grep -q "yahsan2/gh-sub-issue"; then
  use_subissues=true
else
  use_subissues=false
fi
```

统计任务文件数量，选择创建策略：

#### 少于 5 个任务：顺序创建

```bash
for task_file in .claude/epics/$ARGUMENTS/[0-9][0-9][0-9].md; do
  [ -f "$task_file" ] || continue
  task_name=$(grep '^name:' "$task_file" | sed 's/^name: *//' | tr -d '"')
  sed '1,/^---$/d; 1,/^---$/d' "$task_file" > /tmp/task-body.md

  if [ "$use_subissues" = true ]; then
    task_number=$(gh sub-issue create \
      --parent "$epic_number" \
      --title "$task_name" \
      --body-file /tmp/task-body.md \
      --label "task,epic:$ARGUMENTS" \
      --json number -q .number)
  else
    task_number=$(gh issue create \
      --repo "$REPO" \
      --title "$task_name" \
      --body-file /tmp/task-body.md \
      --label "task,epic:$ARGUMENTS" \
      --json number -q .number)
  fi

  echo "$task_file:$task_number" >> /tmp/task-mapping.txt
done
```

#### 5 个或以上：使用 Task tool 并行创建

分批处理，每批 3-4 个任务。

### 3. 重命名任务文件并更新引用

```bash
# 构建旧编号 → 新 Issue ID 的映射
> /tmp/id-mapping.txt
while IFS=: read -r task_file task_number; do
  old_num=$(basename "$task_file" .md)
  echo "$old_num:$task_number" >> /tmp/id-mapping.txt
done < /tmp/task-mapping.txt

# 重命名文件并更新 frontmatter
while IFS=: read -r task_file task_number; do
  new_name="$(dirname "$task_file")/${task_number}.md"
  content=$(cat "$task_file")

  # 更新 depends_on 中的旧编号为新 Issue ID
  while IFS=: read -r old_num new_num; do
    content=$(echo "$content" | sed "s/\b$old_num\b/$new_num/g")
  done < /tmp/id-mapping.txt

  echo "$content" > "$new_name"
  [ "$task_file" != "$new_name" ] && rm "$task_file"

  # 更新 github 字段
  repo=$(gh repo view --json nameWithOwner -q .nameWithOwner)
  github_url="https://github.com/$repo/issues/$task_number"
  current_date=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
  sed -i.bak "/^github:/c\github: $github_url" "$new_name"
  rm "${new_name}.bak"
done < /tmp/task-mapping.txt
```

### 4. 更新 Epic Frontmatter

```bash
repo=$(gh repo view --json nameWithOwner -q .nameWithOwner)
epic_url="https://github.com/$repo/issues/$epic_number"
sed -i.bak "/^github:/c\github: $epic_url" .claude/epics/$ARGUMENTS/epic.md
rm .claude/epics/$ARGUMENTS/epic.md.bak
```

### 5. 创建 Mapping 文件

创建 `.claude/epics/$ARGUMENTS/github-mapping.md`，记录本地文件与 GitHub Issue 的对应关系。

### 6. 输出

```
✓ 已同步到 GitHub
  - Epic: #{epic_number}
  - 任务：{count} 个 sub-issue 已创建
  - 文件已重命名：001.md → {issue_id}.md
  - 引用已更新：depends_on 使用 Issue ID

下一步：
  - 启动 Agent Team：/pm:epic-start $ARGUMENTS
  - 单独处理 Issue：/pm:issue-start {issue_number}
```

## 错误处理

遵循 `rules/github-operations.md` 处理 GitHub CLI 错误。

Issue 创建失败时：
- 报告已成功的部分
- 说明失败原因
- 不回滚（部分同步可接受）
