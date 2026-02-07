# GitHub 操作规则

所有命令中 GitHub CLI 操作的标准模式。

## 关键：仓库保护

**在任何创建/修改 Issue 或 PR 的 GitHub 操作之前**，必须执行：

```bash
remote_url=$(git remote get-url origin 2>/dev/null || echo "")
if [[ "$remote_url" == *"automazeio/pm"* ]] || [[ "$remote_url" == *"automazeio/ccpm"* ]] || [[ "$remote_url" == *"tidbit/ccpm"* ]]; then
  echo "❌ 错误：不能在 PM 模板仓库上执行此操作！"
  echo ""
  echo "此仓库是模板，不应在此创建 Issues 或 PR。"
  echo ""
  echo "修复方法："
  echo "1. Fork 到你自己的 GitHub 账户"
  echo "2. 更新 remote origin："
  echo "   git remote set-url origin https://github.com/YOUR_USERNAME/YOUR_REPO.git"
  echo ""
  echo "当前 remote: $remote_url"
  exit 1
fi
```

此检查**必须**在以下操作前执行：
- 创建 Issue（`gh issue create`）
- 编辑 Issue（`gh issue edit`）
- 评论 Issue（`gh issue comment`）
- 创建 PR（`gh pr create`）
- 任何修改 GitHub 仓库的操作

## 认证

**不预检查认证**，直接执行命令并处理失败：

```bash
gh {command} || echo "❌ GitHub CLI 失败。运行：gh auth login"
```

## 常用操作

### 获取 Issue 详情

```bash
gh issue view {number} --json state,title,labels,body
```

### 创建 Issue

```bash
# 始终指定 repo 避免默认到错误仓库
remote_url=$(git remote get-url origin 2>/dev/null || echo "")
REPO=$(echo "$remote_url" | sed 's|.*github.com[:/]||' | sed 's|\.git$||')
[ -z "$REPO" ] && REPO="user/repo"
gh issue create --repo "$REPO" --title "{title}" --body-file {file} --label "{labels}"
```

### 更新 Issue

```bash
# 先检查 remote origin！
gh issue edit {number} --add-label "{label}"
```

### 添加评论

```bash
# 先检查 remote origin！
gh issue comment {number} --body-file {file}
```

## 错误处理

gh 命令失败时：
1. 明确报错："❌ GitHub 操作失败：{command}"
2. 建议修复："运行 gh auth login 或检查 Issue 编号"
3. 不自动重试

## 重要事项

- **始终**在任何写操作前检查 remote origin
- 信任 gh CLI 已安装并认证
- 使用 `--json` 获取结构化输出
- 保持操作原子性——一个 gh 命令做一件事
- 不预检查 API 限额
