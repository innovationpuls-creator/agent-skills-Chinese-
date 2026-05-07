# agent-skills 入门指南

agent-skills 适用于任何接受 Markdown 指令的 AI 编程代理。本指南涵盖通用方法。如需特定工具的设置，请参见专用指南。

## 技能如何运作

每个技能都是一个 Markdown 文件（`SKILL.md`），描述了特定的工程工作流。当加载到代理的上下文中时，代理会遵循该工作流 —— 包括验证步骤、要避免的反模式，以及退出标准。

**技能不是参考文档。** 它们是代理遵循的分步流程。

## 快速开始（任何代理）

### 1. 克隆仓库

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

### 2. 选择一个技能

浏览 `skills/` 目录。每个子目录包含一个 `SKILL.md`，其中包含：
- **When to use（使用场景）** — 表明此技能适用的触发条件
- **Process（流程）** — 分步工作流
- **Verification（验证）** — 如何确认工作完成
- **Common rationalizations（常见自我合理化）** — 代理可能用来跳过步骤的借口
- **Red flags（危险信号）** — 技能被违反的迹象

### 3. 将技能加载到你的代理中

将相关的 `SKILL.md` 内容复制到代理的系统提示词、规则文件或对话中。最常见的方法：

**系统提示词：** 在会话开始时粘贴技能内容。

**规则文件：** 将技能内容添加到项目的规则文件（CLAUDE.md、.cursorrules 等）中。

**对话中：** 在给出指令时引用技能："对这个变更使用测试驱动开发流程。"

### 4. 使用元技能进行发现

首先加载 `using-agent-skills` 技能。它包含一个流程图，将任务类型映射到相应的技能。

## 推荐设置

### 最小化（从这里开始）

将三个核心技能加载到你的规则文件中：

1. **spec-driven-development** — 用于定义要构建什么
2. **test-driven-development** — 用于证明它能工作
3. **code-review-and-quality** — 用于合并前验证质量

这三个技能覆盖了 AI 辅助开发中最关键的质量缺口。

### 完整生命周期

为了全面覆盖，按阶段加载技能：

```
启动项目：  spec-driven-development → planning-and-task-breakdown
开发期间：  incremental-implementation + test-driven-development
合并前：    code-review-and-quality + security-and-hardening
部署前：    shipping-and-launch
```

### 上下文感知加载

不要一次性加载所有技能 —— 这会浪费上下文。加载与当前任务相关的技能：

- 做 UI 工作？加载 `frontend-ui-engineering`
- 调试？加载 `debugging-and-error-recovery`
- 设置 CI？加载 `ci-cd-and-automation`

## 技能结构

每个技能都遵循相同的结构：

```
YAML frontmatter（name, description）
├── Overview — 这个技能做什么
├── When to Use — 触发条件和场景
├── Core Process — 分步工作流
├── Examples — 代码示例和模式
├── Common Rationalizations — 借口和反驳
├── Red Flags — 技能被违反的迹象
└── Verification — 退出标准检查清单
```

完整的规范请参见 [skill-anatomy.md](skill-anatomy.md)。

## 使用代理

`agents/` 目录包含预配置的角色：

| 代理 | 用途 |
|-------|---------|
| `code-reviewer.md` | 五轴代码审查 |
| `test-engineer.md` | 测试策略和编写 |
| `security-auditor.md` | 漏洞检测 |

当你需要专业审查时加载角色定义。例如，告诉你的编程代理"使用 code-reviewer 角色审查这个变更"并提供角色定义。

## 使用命令

`.claude/commands/` 目录包含 Claude Code 的斜杠命令：

| 命令 | 调用的技能 |
|---------|---------------|
| `/spec` | spec-driven-development |
| `/plan` | planning-and-task-breakdown |
| `/build` | incremental-implementation + test-driven-development |
| `/test` | test-driven-development |
| `/review` | code-review-and-quality |
| `/ship` | shipping-and-launch |

## 使用参考资料

`references/` 目录包含补充检查清单：

| 参考资料 | 配合使用 |
|-----------|----------|
| `testing-patterns.md` | test-driven-development |
| `performance-checklist.md` | performance-optimization |
| `security-checklist.md` | security-and-hardening |
| `accessibility-checklist.md` | frontend-ui-engineering |

当你需要技能覆盖范围之外的详细模式时，加载参考资料。

## 规格和任务产物

`/spec` 和 `/plan` 命令会创建工作产物（`SPEC.md`、`tasks/plan.md`、`tasks/todo.md`）。在工作时将它们视为**活文档**：

- 在开发期间将它们纳入版本控制，这样人类和代理就有共享的事实来源。
- 当范围或决策发生变化时更新它们。
- 如果你的仓库不想长期保留这些文件，在合并前删除它们或将文件夹添加到 `.gitignore` —— 工作流不要求它们永久存在。

## 提示

1. **对于任何非平凡的工作，从 spec-driven-development 开始**
2. **编写代码时始终加载 test-driven-development**
3. **不要跳过验证步骤** —— 它们是关键所在
4. **选择性地加载技能** —— 更多的上下文并不总是更好
5. **使用代理进行审查** —— 不同的视角发现不同的问题
