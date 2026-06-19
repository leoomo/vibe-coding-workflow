# Vibe Coding Workflow

> [English](README.en.md) | 简体中文

> 真正的 vibe coding 不是"给 AI 一句话等它写完"，而是靠一套流程，让 AI 产出**可控、可审查、可验证**。

这是一个 [Claude Code Skill](https://docs.claude.com/en/docs/claude-code/skills)，把"规格驱动开发"固化成一条强制管线：**上下文工程 → 对抗式 Reviewer 子代理 → PRD → 实现文档 → 测试设计 → 分步实现 + 逐步提交 → 验收门禁**。

## 核心铁律

**在被审查的书面计划获批准前，绝不写代码。**

人不必逐行看代码，但必须把注意力放在**流程设计、上下文质量、验收标准、风险控制**上。设计一旦获批准，从建分支、分步实现、逐步自动提交，到验收门禁，全部由 AI 自主跑完——人只在"设计批准"这一处介入一次。

## 为什么需要它

AI 写出来的东西看似逻辑完整，却常藏隐患：需求理解偏差、过度设计、边界遗漏、数据结构不合理、测试不足、不顾线上兼容。本 skill 用三件事对抗这些：

1. **每份关键产出都经"对抗式 Reviewer 子代理"审查**——Writer 推进，Reviewer 挑刺，循环至收敛。
2. **测试与实现同步设计**，并在写代码前先有被批准的规格。
3. **设计决策获批准后，全程自动**。

> 单个 AI 会自洽；让第二个 AI 专门找问题，是这套流程的命门。

## 安装

把整个目录放进 Claude Code 的 skills 目录：

```bash
# 放到全局 skills 目录
git clone https://github.com/leoomo/vibe-coding-workflow.git \
  ~/.claude/skills/vibe-coding-workflow
```

也可以放到某个项目的 `.claude/skills/` 下，仅对该项目生效。安装后在 Claude Code 中即可用 `/vibe-coding-workflow` 调用。

## 用法

```bash
# 交互模式（推荐）
/vibe-coding-workflow <任务描述>

# 指定功能名称
/vibe-coding-workflow <功能名称> <任务描述>
```

| 参数 | 说明 |
|------|------|
| `<功能名称>` | 可选，功能目录名称（kebab-case） |
| `<任务描述>` | 必需，要实现的功能或要修的 bug |

示例：

```bash
/vibe-coding-workflow 用户邀请功能：邀请人发邮件，被邀请人点链接完成注册

/vibe-coding-workflow invite-by-email 修复：邀请链接在某些邮箱客户端被截断，无法点击
```

## 工作流程

每个任务会按以下七步推进：

| 步骤 | 阶段 | 是否人工 |
|------|------|----------|
| Step 0 | 启动、命名与**建独立分支** | 自动 |
| Step 1 | **上下文工程**（Explore agent 深读项目） | 自动 |
| Step 2 | **PRD**（Writer + 对抗式 Reviewer 循环） | 自动 |
| Step 3 | **实现文档 + 测试设计**（Writer + Reviewer 循环） | 自动 |
| Step 4 | **计划批准门禁** | ⚠️ 唯一人工门禁 |
| Step 5 | 分步实现 + **逐步自动提交** | 自动 |
| Step 6 | **验收门禁**（全量测试 / bugfix 双分支验证） | 自动 |
| Step 7 | 总结与记忆确认 | 自动 |

- **Step 4 是唯一需要人介入的点**。确认后，Step 5–7 全自动执行，不再逐步询问。
- bugfix 任务在 Step 6 多一道**双分支验证**：确认修复前测试确实失败、修复后确实通过，证明测试真覆盖了该 bug，而非"碰巧通过"。

## 产出物

每个任务在**目标项目**内产出一套规格文档，便于审查与归档：

```
<项目>/.vibe-coding/<功能名称>/
├── context.md      # 上下文工程产出
├── prd.md          # 做什么（Writer + Reviewer 打磨）
├── impl.md         # 怎么做（实现文档 + 测试设计）
├── review-log.md   # Reviewer 挑刺与处置记录
└── todo.md         # 细粒度任务（每条带测试 + 提交点）
```

> 这些文档生成在你要开发的项目里，而不是本 skill 目录里。

## 规则（刚性，不可跳过）

- **先规格后代码**：批准的 `impl.md` 之前不写任何业务代码。
- **批准后全自动**：Step 4 获批后，Step 5–7 自主执行，不再逐步询问。
- **独立分支**：Step 0 一开始就建分支；所有提交在该分支上，保证可回滚、不污染主干。
- **Reviewer 不可跳过**：PRD、impl 阶段必须经对抗式 Reviewer 子代理审查并收敛。
- **每步带测试 + 自动提交**：不允许"裸代码"步骤，不允许一次性大提交。
- **验收门禁强制**：全量测试须全绿；bugfix 须双分支验证。
- **复用优先**：先在 `context.md` 里找既有实现，再决定新增。
- **测试命令以项目为准**：验收门禁跑哪些命令，按项目实际（见 `context.md`「测试方式」）。

## 仓库结构

```
vibe-coding-workflow/
├── SKILL.md                                  # 技能主入口（流程定义）
├── README.md                                 # 本文件
└── references/
    ├── context-checklist.md                  # Step 1 上下文收集清单
    ├── prd-template.md                       # Step 2 PRD 模板
    ├── implementation-doc-template.md        # Step 3 实现文档 + 测试设计模板
    ├── reviewer-prompt.md                    # Step 2/3 对抗式 Reviewer 提示词
    └── acceptance-checklist.md               # Step 6 验收门禁清单
```

`references/` 下的模板/提示词在对应步骤**按需读取**，不会一次性全塞进上下文。

## 适用场景

- ✅ 实现**新功能**（feature）
- ✅ 修复**复杂 bug**（bugfix，带双分支验证）
- ✅ 任何希望产出**可审查、可验证、可回滚**的 AI 编码任务

不太适合：一行就能改完的拼写修正、或纯探索性、无需落代码的提问。

## 不包含

本 skill **不含合并 / PR 阶段**——验收门禁通过即结束，合并交由用户或其它流程决定。

## License

MIT
