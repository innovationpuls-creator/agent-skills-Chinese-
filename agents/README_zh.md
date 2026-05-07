# 代理角色

扮演单一角色、单一视角的专家角色。每个角色都是一个 Markdown 文件，作为系统提示词被你的 harness（Claude Code、Cursor、Copilot 等）消费。

| 角色 | 角色定位 | 最适合场景 |
|---------|------|----------|
| [code-reviewer](code-reviewer.md) | 资深技术专家 | 合并前的五轴审查 |
| [security-auditor](security-auditor.md) | 安全工程师 | 漏洞检测、OWASP 风格审计 |
| [test-engineer](test-engineer.md) | QA 工程师 | 测试策略、覆盖率分析、Prove-It 模式 |

## 角色与技能、命令的关系

三个层级，各有分工：

| 层级 | 是什么 | 示例 | 组合角色 |
|-------|-----------|---------|------------------|
| **技能** | 带步骤和退出标准的工作流 | `code-review-and-quality` | *方法* — 从角色或命令内部调用 |
| **角色** | 有视角和输出格式的角色 | `code-reviewer` | *谁* — 采用视角，产出报告 |
| **命令** | 用户入口点 | `/review`, `/ship` | *何时* — 组合角色和技能 |

用户（或斜杠命令）是编排者。**角色不调用其他角色。** 技能是角色工作流中的必要环节。

## 何时使用

### 直接调用角色
当你想要对当前变更获得一个视角，且用户在循环中时选择此项。

- "审查这个 PR" → 直接调用 `code-reviewer`
- "`auth.ts` 中有安全问题吗？" → 直接调用 `security-auditor`
- "结账流程缺少哪些测试？" → 直接调用 `test-engineer`

### 斜杠命令（背后是单一角色）
当你有可重复的工作流、每次都要重新解释时选择此项。

- `/review` → 用项目的审查技能包装 `code-reviewer`
- `/test` → 用 TDD 技能包装 `test-engineer`

### 斜杠命令（编排者 — 并行发散）
**只有当**独立的调查可以并行运行并产生报告，然后由单一代理合并时，才选择此项。

- `/ship` → 并行发散到 `code-reviewer` + `security-auditor` + `test-engineer`，然后将他们的报告合成为 go/no-go 决策

这是本仓库唯一认可的编排模式。完整的模式目录和反模式请参见 [references/orchestration-patterns.md](../references/orchestration-patterns.md)。

## 决策矩阵

```
工作是对单个工件的单视角分析吗？
├── 是 → 直接调用角色
└── 否  → 子任务是独立的（无共享可变状态，无排序）？
         ├── 是 → 带并行发散的斜杠命令（如 /ship）
         └── 否 → 由用户顺序运行斜杠命令（/spec → /plan → /build → /test → /review）
```

## 工作示例：有效的编排

`/ship` 是本仓库中典型的并行发散编排器：

```
/ship
  ├── (并行) code-reviewer    → 审查报告
  ├── (并行) security-auditor → 审计报告
  └── (并行) test-engineer    → 覆盖率报告
                  ↓
        合并阶段（主代理）
                  ↓
        go/no-go 决策 + 回滚计划
```

为什么这样有效：
- 每个子代理操作同一份 diff，但产出**不同的视角**
- 它们之间没有依赖 → 真正的并行，节省真实时钟时间
- 每个都在新的上下文窗口中运行 → 主会话保持整洁
- 合并步骤很小且受益于完整上下文，所以留在主代理中

## 工作示例：无效的编排（不要这样构建）

一个 `meta-orchestrator` 角色的工作是"决定调用哪个其他角色"：

```
/work-on-pr → meta-orchestrator
                  ↓ (决定"这需要审查")
              code-reviewer
                  ↓ (返回)
              meta-orchestrator (转述结果)
                  ↓
              user
```

为什么这样会失败：
- 纯路由层，无领域价值
- 添加两个转述跳转 → 信息丢失 + 2× token 成本
- 用户已经知道他们想要审查；让他们直接调用 `/review`
- 复制了斜杠命令和 `AGENTS.md` 意图映射已经做的工作

## 角色规则

1. 角色是单一角色，具有单一输出格式。如果你发现自己添加了第二个角色，就创建第二个角色。
2. **角色不调用其他角色。** 组合是斜杠命令或用户的职责。在 Claude Code 上这也是一个硬性平台约束 —— *"子代理不能生成其他子代理"* —— 所以这条规则是强制执行的。
3. 角色可以调用技能（*方法*）。
4. 每个角色文件以一个 **Composition** 块结尾，说明它适合的位置。

## Claude Code 互操作

本仓库中的角色设计为可作为 Claude Code 子代理和 Agent Teams 队友工作，无需修改：

- **作为子代理：** 当启用此插件时自动发现（无需路径配置）。使用 Agent 工具和 `subagent_type: code-reviewer`（或 `security-auditor`、`test-engineer`）。`/ship` 是典型示例。
- **作为 Agent Teams 队友**（实验性，需要 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`）：在生成队友时引用相同的角色名称。角色的内容**追加到**队友的系统提示词中作为额外指令（不是替换），所以你的角色文本位于主代理安装的团队协调指令之上（SendMessage、task-list 工具等）。

子代理只向主代理报告结果。Agent Teams 让队友直接相互发送消息。当报告足够时使用子代理；当子代理需要相互挑战发现结果时使用 Agent Teams（例如竞争假设调试）。完整的映射请参见 [references/orchestration-patterns.md](../references/orchestration-patterns.md)。

插件代理不支持 `hooks`、`mcpServers` 或 `permissionMode` frontmatter —— 这些字段会被静默忽略。在创作新角色时避免依赖它们。

## 添加新角色

1. 使用与现有角色相同的 frontmatter 格式创建 `agents/<role>.md`。
2. 定义角色、范围、输出格式和规则。
3. 在底部添加 **Composition** 块（直接调用时机 / 通过什么调用 / 不可从其他角色调用）。
4. 将角色添加到本文件顶部的表格中。
5. 如果角色启用了新的编排模式，在 `references/orchestration-patterns.md` 中记录它，而不是在角色文件本身中发明模式。
