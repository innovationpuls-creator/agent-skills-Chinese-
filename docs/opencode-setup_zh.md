# OpenCode 设置

本指南解释如何以与 Claude Code 体验紧密匹配的方式在 OpenCode 中使用 Agent Skills（自动技能选择、生命周期驱动的工作流和严格的流程执行）。

## 概述

OpenCode 支持自定义 `/commands`，但没有类似 Claude Code 的原生插件系统或自动技能路由。

相反，我们通过以下方式实现对等：

- 强大的系统提示词（`AGENTS.md`）
- 内置的 `skill` 工具
- 从 `/skills` 目录进行一致的技能发现

这创建了一种**代理驱动的工作流**，技能被自动选择和执行。

虽然可以在 OpenCode 中重新创建 `/spec`、`/plan` 和其他命令，但此集成有意使用代理驱动的方法：

- 技能根据意图自动选择
- 工作流通过 `AGENTS.md` 强制执行
- 无需手动调用命令

这更接近 Claude Code 在实践中的行为方式，技能自动触发而非手动。

---

## 安装

1. 克隆仓库：

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

2. 在 OpenCode 中打开项目。

3. 确保以下文件存在于你的工作区：

- `AGENTS.md`（根目录）
- `skills/` 目录

无需额外安装。

---

## 工作原理

### 1. 技能发现

所有技能位于：

```
skills/<skill-name>/SKILL.md
```

OpenCode 代理被指示（通过 `AGENTS.md`）：

- 检测技能何时适用
- 调用 `skill` 工具
- 严格遵循技能

### 2. 自动技能调用

代理评估每个请求并将其映射到适当的技能。

示例：

- "构建功能" → `incremental-implementation` + `test-driven-development`
- "设计系统" → `spec-driven-development`
- "修复 bug" → `debugging-and-error-recovery`
- "审查代码" → `code-review-and-quality`

用户**不需要**显式请求技能。

### 3. 生命周期映射（隐式命令）

开发生命周期被隐式编码：

- DEFINE → `spec-driven-development`
- PLAN → `planning-and-task-breakdown`
- BUILD → `incremental-implementation` + `test-driven-development`
- VERIFY → `debugging-and-error-recovery`
- REVIEW → `code-review-and-quality`
- SHIP → `shipping-and-launch`

这取代了 `/spec`、`/plan` 等斜杠命令。

---

## 使用示例

### 示例 1：功能开发

用户：
```
为这个应用添加身份验证
```

代理行为：
- 检测功能工作
- 调用 `spec-driven-development`
- 在写代码之前生成规格
- 进入规划和实现技能

---

### 示例 2：Bug 修复

用户：
```
这个端点返回 500 错误
```

代理行为：
- 调用 `debugging-and-error-recovery`
- 复现 → 定位 → 修复 → 添加防护

---

### 示例 3：代码审查

用户：
```
审查这个 PR
```

代理行为：
- 调用 `code-review-and-quality`
- 应用结构化审查（正确性、设计、可读性等）

---

## 代理期望（关键）

为了让 OpenCode 正确工作，代理必须遵循这些规则：

- 在行动前始终检查技能是否适用
- 如果技能适用，必须使用
- 永远不要跳过必需的工作流（规格、计划、测试等）
- 不要直接跳到实现

这些规则通过 `AGENTS.md` 强制执行。

---

## 限制

- 没有原生斜杠命令（改为通过意图映射处理）
- 没有插件系统（通过提示词 + 结构处理）
- 技能调用取决于模型合规性

尽管如此，工作流在实践中与 Claude Code 紧密匹配。

---

## 推荐工作流

直接使用自然语言：

- "设计一个功能"
- "规划这个变更"
- "实现这个"
- "修复这个 bug"
- "审查这个"

代理会自动选择并执行正确的技能。

---

## 总结

OpenCode 集成通过组合实现：

- 结构化技能（本仓库）
- 强大的代理规则（`AGENTS.md`）
- 通过推理的自动技能调用

这产生了一个**完全代理驱动的、生产级工程工作流**，无需插件或手动命令。
