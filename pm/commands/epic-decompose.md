---
allowed-tools: Bash, Read, Write, LS, Task
---

# Epic Decompose

将 Epic 拆分为具体可执行的任务。

## 用法

```
/pm:epic-decompose <feature_name>
```

## 前置检查

静默完成以下检查，不向用户报告进度。

1. **验证 Epic 存在**：`.claude/epics/$ARGUMENTS/epic.md` 不存在时："✗ Epic 未找到。先运行：/pm:prd-parse $ARGUMENTS"
2. **检查已有任务**：若已存在编号任务文件，询问是否删除重建
3. **验证 Epic Frontmatter**：必须包含 type, name, status, created, prd
4. **检查 Epic 状态**：若 status 为 done，警告用户

## 指令

你正在为 **$ARGUMENTS** 拆分具体任务。

### 1. 读取 Epic

- 加载 `.claude/epics/$ARGUMENTS/epic.md`
- 理解技术方案和需求
- 分析任务分解预览

### 2. 并行创建策略

根据任务数量选择创建方式：

- **< 5 个任务**：顺序创建
- **5-10 个任务**：分 2-3 批次，使用 Task tool 并行创建

并行创建示例：
```yaml
Task:
  description: "创建任务文件批次 {X}"
  subagent_type: "general-purpose"
  prompt: |
    为 epic $ARGUMENTS 创建任务文件。
    任务列表：{具体任务}
    文件路径：.claude/epics/$ARGUMENTS/{number}.md
    使用指定的 frontmatter 和内容格式。
```

### 3. 任务文件格式

遵循 `rules/frontmatter-spec.md` 的 Task Frontmatter 规范。

每个任务文件 `.claude/epics/$ARGUMENTS/{number}.md`：

```markdown
---
type: task
name: "任务标题中文"
status: open
created: [运行 date -u +"%Y-%m-%dT%H:%M:%SZ" 获取真实时间]
github: ""
depends_on: []
parallel: true
---

# 任务：{任务标题}

## 描述
清晰简洁的任务说明

## 验收标准（6.031 方法论）

### 功能验收
- [ ] 具体标准 1
- [ ] 具体标准 2

### 等价类测试
- [ ] 正常输入：{描述}
- [ ] 边界值：{描述}
- [ ] 异常输入：{描述}

## 技术细节
- 实现方案
- 关键考量
- 涉及的文件

## 依赖
- 任务依赖
- 外部依赖

## 工作量估算
- 规模：XS / S / M / L / XL

## 完成定义（Definition of Done）
- [ ] 代码实现
- [ ] 规范文档（前置/后置条件）
- [ ] 测试编写并通过（等价类 + 边界值）
- [ ] checkRep 验证（如涉及 ADT）
- [ ] 代码审查通过
```

### 4. 任务命名

- 文件名：`001.md`, `002.md`, ...（顺序编号）
- sync 后会重命名为 GitHub Issue 编号

### 5. 依赖验证

- 确保引用的依赖任务存在
- 检查无循环依赖
- 发现问题时警告但继续

### 6. 更新 Epic

创建完所有任务后：

1. 在 epic.md 中添加任务汇总：

```markdown
## 任务汇总
- [ ] 001 - {任务标题} (parallel: true/false)
- [ ] 002 - {任务标题} (parallel: true/false)

总任务数：{count}
可并行：{parallel_count}
需顺序：{sequential_count}
```

2. **更新 Epic Frontmatter 的 `task_count` 字段**为实际创建的任务数

### 7. 完成后

1. 确认："✓ 为 $ARGUMENTS 创建了 {count} 个任务"
2. 总结：总任务数、可并行/需顺序的比例
3. 建议下一步："运行 `/pm:epic-sync $ARGUMENTS` 同步到 GitHub"
