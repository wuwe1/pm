---
allowed-tools: Bash, Read, Write, LS
---

# PRD Parse

将 PRD 转化为技术实现 Epic。

## 用法

```
/pm:prd-parse <feature_name>
```

## 前置检查

静默完成以下检查，不向用户报告进度。

1. **验证参数**：未提供 feature_name 时报错
2. **验证 PRD 存在**：`.claude/prds/$ARGUMENTS.md` 不存在时："❌ PRD 未找到。先运行：/pm:prd-new $ARGUMENTS"
3. **验证 PRD Frontmatter**：必须包含 type, name, description, status, created
4. **检查已有 Epic**：若 `.claude/epics/$ARGUMENTS/epic.md` 已存在，询问是否覆盖
5. **确保目录存在**：`mkdir -p .claude/epics/$ARGUMENTS`

## 指令

你是一位技术负责人，将 PRD 转化为技术实现计划。目标是 **$ARGUMENTS**。

### 1. 读取 PRD

- 加载 `.claude/prds/$ARGUMENTS.md`
- 分析所有需求、约束和用户故事
- 提取设计契约

### 2. 技术分析

#### 架构决策
- 关键技术选型和理由
- 设计模式选择

#### ADT 设计考量（6.031）
- 识别核心数据抽象
- 定义操作分类（Creator / Producer / Observer / Mutator）
- 确保表示独立性

#### 技术方案
- 前端组件（如适用）
- 后端服务（如适用）
- 基础设施（如适用）

#### 实现策略
- 开发阶段划分
- 风险缓解

#### 测试方案（6.031 方法论）
- 等价类划分策略
- 边界值测试计划
- 回归测试策略
- 目标覆盖率

#### 任务分解预览
高层次任务类别（后续由 epic-decompose 详细拆分）

**重要**：任务总数限制在 **10 个以内**。寻找简化方案，优先复用已有功能。

### 3. 文件格式

遵循 `rules/frontmatter-spec.md` 的 Epic Frontmatter 规范。

创建 `.claude/epics/$ARGUMENTS/epic.md`：

```markdown
---
type: epic
name: $ARGUMENTS
status: backlog
created: [运行 date -u +"%Y-%m-%dT%H:%M:%SZ" 获取真实时间]
prd: .claude/prds/$ARGUMENTS.md
github: ""
task_count: 0
tasks_done: 0
---

# Epic: $ARGUMENTS

## 概述
技术方案简述

## 架构决策
- 关键决策和理由

## ADT 设计
- 核心数据抽象定义
- 操作分类
- 表示独立性保证

## 技术方案
### 前端
[内容]
### 后端
[内容]
### 基础设施
[内容]

## 实现策略
- 开发阶段
- 风险缓解

## 测试方案
- 等价类划分策略
- 边界值测试
- 覆盖率目标

## 任务分解预览
- [ ] 类别 1：描述
- [ ] 类别 2：描述

## 依赖
- 外部依赖
- 内部依赖

## 成功标准（技术）
- 性能基准
- 质量关卡
```

### 4. 质量检查

保存前验证：
- PRD 所有需求都在技术方案中有对应
- 任务分解类别覆盖所有实现领域
- 任务数 ≤ 10
- 架构决策有理由支撑

### 5. 完成后

1. 确认："✅ Epic 已创建：.claude/epics/$ARGUMENTS/epic.md"
2. 总结：任务类别数、关键架构决策
3. 建议下一步："运行 `/pm:epic-decompose $ARGUMENTS` 拆分为具体任务"
