# 在 Windsurf 中使用 agent-skills

## 设置

### 项目规则

Windsurf 使用 `.windsurfrules` 进行项目特定的代理指令：

```bash
# 从你最重要的技能创建一个组合规则文件
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md >> .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .windsurfrules
```

### 全局规则

对于你想在所有项目中使用的技能，将它们添加到 Windsurf 的全局规则：

1. 打开 Windsurf → 设置 → AI → 全局规则
2. 粘贴你最常用技能的内容

## 推荐配置

将 `.windsurfrules` 聚焦于 2-3 个核心技能，以保持在上下文限制内：

```
# .windsurfrules
# 本项目的核心 agent-skills

[粘贴 test-driven-development SKILL.md]

---

[粘贴 incremental-implementation SKILL.md]

---

[粘贴 code-review-and-quality SKILL.md]
```

## 使用提示

1. **有选择性** — Windsurf 的上下文有限。选择解决你最大质量缺口的技能。
2. **在对话中引用** — 在特定阶段工作时，将额外技能内容粘贴到聊天中（例如，在构建身份验证时粘贴 `security-and-hardening`）。
3. **使用参考资料作为检查清单** — 粘贴 `references/security-checklist.md` 并让 Windsurf 验证每个项目。
