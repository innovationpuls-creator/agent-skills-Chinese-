# 在 Cursor 中使用 agent-skills

## 设置

### 选项 1：规则目录（推荐）

Cursor 支持 `.cursor/rules/` 目录用于项目特定规则：

```bash
# 创建规则目录
mkdir -p .cursor/rules

# 复制你想要的技能作为规则
cp /path/to/agent-skills/skills/test-driven-development/SKILL.md .cursor/rules/test-driven-development.md
cp /path/to/agent-skills/skills/code-review-and-quality/SKILL.md .cursor/rules/code-review-and-quality.md
cp /path/to/agent-skills/skills/incremental-implementation/SKILL.md .cursor/rules/incremental-implementation.md
```

此目录中的规则会自动加载到 Cursor 的上下文中。

### 选项 2：.cursorrules 文件

在项目根目录创建一个 `.cursorrules` 文件，内联核心技能：

```bash
# 生成组合规则文件
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .cursorrules
echo "\n---\n" >> .cursorrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .cursorrules
```

## 推荐配置

### 核心技能（始终加载）

添加到 `.cursor/rules/`：

1. `test-driven-development.md` — TDD 工作流和 Prove-It 模式
2. `code-review-and-quality.md` — 五轴审查
3. `incremental-implementation.md` — 以小的可验证切片构建

### 阶段特定技能（按需加载）

对于特定阶段的工作，根据需要创建额外的规则文件：

- `spec-development.md` -> `spec-driven-development/SKILL.md`
- `frontend-ui.md` -> `frontend-ui-engineering/SKILL.md`
- `security.md` -> `security-and-hardening/SKILL.md`
- `performance.md` -> `performance-optimization/SKILL.md`

在处理相关任务时将这些添加到 `.cursor/rules/`，完成后删除以管理上下文限制。

## 使用提示

1. **不要一次性加载所有技能** - Cursor 有上下文限制。作为规则加载 2-3 个核心技能，并根据需要添加阶段特定技能。
2. **明确引用技能** - 告诉 Cursor"对这个变更遵循 test-driven-development 规则"以确保它阅读加载的规则。
3. **使用代理进行审查** - 复制 `agents/code-reviewer.md` 内容，告诉 Cursor"使用这个代码审查框架审查这个 diff"。
4. **按需加载参考资料** - 在处理性能工作时，将 `performance.md` 添加到 `.cursor/rules/` 或直接粘贴检查清单内容。
