# 需求分级与流程选择规则

> 本文档定义了基于需求复杂度的三种开发流程模式，Agent 在接收到需求时必须根据此规则自动选择并推荐对应的流程模式。

---

## 分级标准

### 判断维度

| 维度 | 说明 |
|------|------|
| **影响范围** | 涉及多少模块/层级/文件 |
| **架构影响** | 是否改变系统架构、新增抽象层、引入新模式 |
| **不确定性** | 需求是否明确，实现路径是否唯一 |
| **协作需求** | 是否涉及多人协作、多 PR 协调 |
| **回退风险** | 错误实现的回退成本高低 |

### 分级矩阵

| 等级 | 条件（满足任一组即可判定） |
|------|--------------------------|
| **复杂** | ① 涉及 3+ 模块且架构变更<br>② 需求不明确，需多轮澄清<br>③ 多人协作功能<br>④ 回退风险高（数据库迁移、API 契约变更） |
| **一般** | ① 涉及 1-2 模块，清晰的增删改<br>② 需求明确，实现路径唯一或有限<br>③ 单人可完成，不影响全局架构 |
| **简易** | ① 单文件或 2 文件以内的小改动<br>② 需求完全明确，无歧义<br>③ hotfix、配置调整、文案修改 |

---

## 三种流程模式

### 模式一：复杂模式（Complex Mode）

> 适用场景：大型功能、架构变更、多人协作功能。每步必须严格验证后才可进入下一步。

#### 流程图

```
阶段 1: 探索与需求
  ① skill: superpowers/using-superpowers
  ② skill: superpowers/brainstorming
     └→ 循环：提问 → 方案对比 → 逐段确认 → 设计文档
  ③ /opsx:explore（可选，代码库预调研）
  ✅ 门控：用户审批设计方案后才可继续

阶段 2: 规划制品（逐个创建 + 逐个验证）
  ④ /opsx:new <change-name>
  ⑤ /opsx:continue → proposal
     └→ 验证: openspec validate <change>
  ⑥ /opsx:continue → specs
     └→ 验证: openspec validate <change>
  ⑦ /opsx:continue → design
     └→ 验证: openspec validate <change>
  ⑧ /opsx:continue → tasks
     └→ 验证: openspec validate <change>
  ✅ 门控：所有制品 validate 通过

阶段 3: 工作空间隔离
  ⑨ skill: superpowers/using-git-worktrees
     └→ 验证: 测试基线通过

阶段 4: 实施计划
  ⑩ skill: superpowers/writing-plans
     └→ 保存至 docs/superpowers/plans/
     └→ 验证: 每步有精确文件路径、完整代码、预期输出

阶段 5: 逐任务执行（循环）
  对每个任务:
  ⑪ skill: superpowers/subagent-driven-development
     ├→ 派发实现者子 Agent
     │  └→ skill: superpowers/test-driven-development
     ├→ skill: superpowers/verification-before-completion
     ├→ skill: superpowers/requesting-code-review（规格符合性审查）
     ├→ skill: superpowers/receiving-code-review
     ├→ skill: superpowers/requesting-code-review（代码质量审查）
     └→ skill: superpowers/receiving-code-review
  ✅ 门控：双审查通过后才标记任务完成

阶段 6: 收尾
  ⑫ skill: superpowers/finishing-a-development-branch
  ⑬ /opsx:verify（三维度验证：完整/正确/一致）
  ⑭ /opsx:sync（delta specs 合并到主 specs）
  ⑮ /opsx:archive
```

#### 命令链路

```
using-superpowers
  → brainstorming
  → [/opsx:explore]                          ← 可选
  → /opsx:new <change-name>
  → /opsx:continue (proposal) + openspec validate
  → /opsx:continue (specs)    + openspec validate
  → /opsx:continue (design)   + openspec validate
  → /opsx:continue (tasks)    + openspec validate
  → using-git-worktrees
  → writing-plans
  → subagent-driven-development (×N)
      → test-driven-development
      → verification-before-completion
      → requesting-code-review (规格)
      → receiving-code-review
      → requesting-code-review (质量)
      → receiving-code-review
  → finishing-a-development-branch
  → /opsx:verify
  → /opsx:sync
  → /opsx:archive
```

| 阶段 | Skills | Commands |
|------|--------|----------|
| 需求 | `using-superpowers`、`brainstorming` | `/opsx:explore`（可选） |
| 制品 | — | `/opsx:new`、`/opsx:continue`（循环 ×4，每步 `openspec validate`） |
| 隔离 | `using-git-worktrees` | — |
| 计划 | `writing-plans` | — |
| 实施 | `subagent-driven-development`、`test-driven-development`、`verification-before-completion`、`requesting-code-review`、`receiving-code-review`（各 ×N） | — |
| 收尾 | `finishing-a-development-branch` | `/opsx:verify`、`/opsx:sync`、`/opsx:archive` |

**最少调用：Skills 11 次 / Commands 7+ 次**

---

### 模式二：一般模式（Normal Mode）

> 适用场景：日常功能开发、bug 修复、小中型需求。保留结构化流程但合并制品创建步骤。

#### 流程图

```
阶段 1: 需求理解
  ① skill: superpowers/using-superpowers
  ② skill: superpowers/brainstorming（缩减版）
     └→ 聚焦：明确目标 + 关键约束，跳过深度方案对比

阶段 2: 一步创建制品
  ③ /opsx:propose <description>
     └→ 验证: openspec validate <change>

阶段 3: 实施计划 + 执行
  ④ skill: superpowers/writing-plans
  ⑤ skill: superpowers/executing-plans
     └→ 每个任务内:
        ├→ skill: superpowers/test-driven-development
        └→ skill: superpowers/verification-before-completion
  ✅ 阶段验证：全部任务完成后
  ⑥ skill: superpowers/verification-before-completion（整体验证）

阶段 4: 收尾
  ⑦ skill: superpowers/finishing-a-development-branch
  ⑧ /opsx:archive
```

#### 命令链路

```
using-superpowers
  → brainstorming (缩减版)
  → /opsx:propose <description> + openspec validate
  → writing-plans
  → executing-plans
      → test-driven-development (每任务)
      → verification-before-completion (每任务)
  → verification-before-completion (整体验证)
  → finishing-a-development-branch
  → /opsx:archive
```

| 阶段 | Skills | Commands |
|------|--------|----------|
| 需求 | `using-superpowers`、`brainstorming`（缩减） | — |
| 制品 | — | `/opsx:propose`（+ `openspec validate`） |
| 计划 | `writing-plans` | — |
| 实施 | `executing-plans`、`test-driven-development`、`verification-before-completion`（各 ×N） | — |
| 收尾 | `finishing-a-development-branch` | `/opsx:archive` |

**最少调用：Skills 7 次 / Commands 3 次**

---

### 模式三：简易模式（Simple Mode）

> 适用场景：小改动、hotfix、单文件修改。用 `/opsx:propose` 一步创建制品（AI 自动生成，人类无需审查），实现后归档。全程约 30 秒额外开销。

#### 流程图

```
阶段 1: 快速确认 + 制品创建
  ① skill: superpowers/using-superpowers
  ② 简短对话确认目标（不调用 brainstorming，直接明确改什么、改哪里）
  ③ /opsx:propose <简短描述>
     └→ AI 自动生成所有制品（proposal/specs/design/tasks），人类无需审查或编辑

阶段 2: 直接实施
  ④ skill: superpowers/test-driven-development
     ├→ RED:   写失败测试 → 验证失败
     ├→ GREEN: 写最小实现 → 验证通过
     └→ REFACTOR: 清理（可选）

阶段 3: 验证 + 归档
  ⑤ skill: superpowers/verification-before-completion
  ⑥ /opsx:archive
  ⑦ 提交（用户明确要求时）
```

#### 命令链路

```
using-superpowers
  → /opsx:propose <简短描述>
  → test-driven-development
  → verification-before-completion
  → /opsx:archive
```

| 阶段 | Skills | Commands |
|------|--------|----------|
| 确认 | `using-superpowers` | — |
| 制品 | — | `/opsx:propose` |
| 实施 | `test-driven-development` | — |
| 收尾 | `verification-before-completion` | `/opsx:archive` |

**最少调用：Skills 3 次 / Commands 2 次**

---

## 速查对比表

| 维度 | 复杂模式 | 一般模式 | 简易模式 |
|------|----------|----------|----------|
| **OpenSpec 制品** | 逐个创建 + 逐个验证 | `/opsx:propose` 一步创建 | `/opsx:propose` 一步创建（无需审查） |
| **需求澄清** | `brainstorming` 完整流程 | `brainstorming` 缩减版 | 简短对话 |
| **工作空间隔离** | `using-git-worktrees` | 按需（可选） | 不使用 |
| **计划编写** | `writing-plans` | `writing-plans` | 不使用 |
| **执行方式** | `subagent-driven-development` | `executing-plans` | 直接执行 |
| **TDD** | 每个子 Agent 内执行 | 每个任务内执行 | 直接执行 |
| **代码审查** | 双审查（规格 + 质量） | 无 | 无 |
| **完成验证** | 每任务 + 整体 + `/opsx:verify` | 整体验证 | 单次验证 |
| **收尾** | `finishing-branch` + `sync` + `archive` | `finishing-branch` + `archive` | `archive` |
| **适用规模** | 大型功能 / 架构变更 | 日常功能 / bug 修复 | hotfix / 单文件改动 |
| **最少 skills 调用** | 11 | 7 | 3 |
| **最少 commands 调用** | 7+ | 3 | 2 |

---

## 核心原则

**三种模式通用：修改一步，验证一步。**

- 复杂模式：每步产出制品 → `openspec validate` → 通过后才下一步
- 一般模式：每步执行 → `verification-before-completion` → 通过后才下一步
- 简易模式：RED → 验证失败 → GREEN → 验证通过 → `/opsx:archive` 归档
