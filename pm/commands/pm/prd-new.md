---
allowed-tools: Bash, Read, Write, LS
---

# PRD New

创建新的产品需求文档（PRD）。

## 用法

```
/pm:prd-new <feature_name>
```

## 前置检查

静默完成以下检查，不向用户报告进度。

1. **验证 feature name 格式**：必须是 kebab-case（小写字母、数字、连字符），以字母开头
   - 无效时："❌ Feature name 必须是 kebab-case 格式。示例：user-auth, payment-v2"

2. **检查已有 PRD**：若 `.claude/prds/$ARGUMENTS.md` 已存在，询问是否覆盖

3. **确保目录存在**：`mkdir -p .claude/prds`

## 指令

你是一位产品经理，为 **$ARGUMENTS** 创建全面的产品需求文档。

### 1. 头脑风暴对话

- 询问澄清问题，了解 feature 的背景和目标
- 识别目标用户和使用场景
- 探索边界情况和约束条件
- **不要跳过这一步**——充分讨论后再写文档

### 2. PRD 结构

#### 概要（Executive Summary）
- 一段话概述价值主张

#### 问题陈述（Problem Statement）
- 要解决什么问题？为什么现在要做？

#### 用户故事（User Stories）

采用 6.031 规范思维，每个故事包含：

```markdown
### 用户故事 {N}：{标题}

**角色**：作为 {角色}
**需求**：我想要 {功能}
**目的**：以便 {价值}

**前置条件**：
- {调用前必须满足的条件}

**后置条件**：
- {成功执行后保证的状态}

**验收标准**：
- [ ] {可验证的标准}
```

#### 需求（Requirements）

**功能需求**
- 核心功能和能力
- 用户交互流程

**非功能需求**
- 性能、安全、可扩展性

**设计契约**
- 模块之间的接口契约
- 数据格式和验证规则
- 错误处理约定

#### 成功标准（Success Criteria）
- 可量化的结果指标

#### 约束与假设
- 技术限制、时间约束

#### 不在范围内（Out of Scope）
- 明确列出不做什么

#### 依赖
- 外部和内部依赖

### 3. 文件格式

遵循 `rules/frontmatter-spec.md` 的 PRD Frontmatter 规范。

保存到 `.claude/prds/$ARGUMENTS.md`：

```markdown
---
type: prd
name: $ARGUMENTS
description: "一行中文简述"
status: backlog
created: [运行 date -u +"%Y-%m-%dT%H:%M:%SZ" 获取真实时间]
---

# PRD: $ARGUMENTS

## 概要
[内容]

## 问题陈述
[内容]

## 用户故事
[内容，每个故事包含前置/后置条件]

## 需求
### 功能需求
[内容]
### 非功能需求
[内容]
### 设计契约
[内容]

## 成功标准
[内容]

## 约束与假设
[内容]

## 不在范围内
[内容]

## 依赖
[内容]
```

### 4. 质量检查

保存前验证：
- 所有章节已完成（无占位符）
- 用户故事包含前置/后置条件和验收标准
- 成功标准可量化
- 不在范围内已明确列出

### 5. 完成后

1. 确认："✅ PRD 已创建：.claude/prds/$ARGUMENTS.md"
2. 简要总结内容
3. 建议下一步："运行 `/pm:prd-parse $ARGUMENTS` 创建技术实现 Epic"
