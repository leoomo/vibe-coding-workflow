# 最终分支审核

> Step 6.5 按需读取。这是 vibe workflow 内嵌的轻量双轴审核，不调用完整 `$code-review`，不搜索 Issue 或重新发现规格。

## 1. 锁定输入

Git 项目先从 `review-log.md` 读取 Step 0 记录的值并设为 `BASE_SHA`，再执行：

```bash
git rev-parse "$BASE_SHA"
git diff "$BASE_SHA"...HEAD
git log "$BASE_SHA"..HEAD --oneline
```

- fixed point 无法解析或 diff 为空：停止并报告，不把错误丢给 Reviewer。
- 非 Git 项目：以 `todo.md` 记录的完整变更文件清单为范围，将路径列表传给两个 Reviewer，并在 `review-log.md` 注明无 commit diff。审核通过后记录所有已审非流程文档的路径 + SHA-256 `FINAL_REVIEW_MANIFEST`。
- Spec 固定为当前任务的 `prd.md`、`impl.md` 及其验收标准。
- Standards 源为 Step 1 已发现的项目规范，如 `AGENTS.md`、`CONTRIBUTING.md`、`CODING_STANDARDS.md`。

## 2. 并行审核

在同一次 Agent 调用中启动两个独立的 `general-purpose` Reviewer，不共享对方结论。

### Standards Reviewer prompt

发送下方 prompt 时，必须把后面的 Smell baseline 逐条完整附上；Reviewer 不能假设自己可以读取本模板。

```text
你是 Standards Reviewer。审核给定的整条分支 diff 和 commit 列表。

输入：
- diff 命令或非 Git 变更文件列表
- commit 列表
- 项目规范文件路径及内容
- 下方 smell baseline

报告每个有证据的问题：
[Critical|Important|Minor] <file:line> - <问题>
证据：<规范条款或具体 diff>
影响：<为什么它会造成错误或显著维护成本>
建议：<最小修复>

硬规则：
- 项目书面规范优先于 smell baseline。
- smell 只能标为 judgement call，不得冒充硬性违规。
- 跳过 lint/typecheck/formatter 已机械执行的问题。
- 只报告本分支引入且可行动的问题；无证据不报。
- 无问题时明确输出「No findings」。限 400 字。
```

Smell baseline（均为判断性启发）：

- **Mysterious Name**：命名不能揭示用途。
- **Duplicated Code**：变更中重复同一逻辑形状。
- **Feature Envy**：方法过度操作其他对象的数据。
- **Data Clumps**：同一组字段或参数反复成组出现。
- **Primitive Obsession**：原始值代替了值得建模的领域概念。
- **Repeated Switches**：同一类型的分支判断在多处重复。
- **Shotgun Surgery**：一个逻辑变更迫使多文件分散修改。
- **Divergent Change**：同一模块在本变更中承担多个无关原因。
- **Speculative Generality**：为规格不需要的未来可能性增加抽象或扩展点。
- **Message Chains**：调用方依赖过长对象导航链。
- **Middle Man**：类或函数大部分只是转发。
- **Refused Bequest**：子类或实现者忽略了大部分继承契约。

### Spec Reviewer prompt

```text
你是 Spec Reviewer。审核给定的整条分支 diff 和 commit 列表。

唯一规格源：当前任务的 prd.md、impl.md 及其验收标准。不搜索 Issue，不用其他文档替代它们。

检查：
- 规格要求但遗漏或只完成一部分的行为。
- 实现了但方式错误的要求。
- 没有规格依据的行为或范围扩张。

报告每个有证据的问题：
[Critical|Important|Minor] <file:line> - <问题>
规格：<prd/impl 的具体条款>
证据：<具体 diff 或缺失位置>
影响：<用户或系统后果>
建议：<最小修复>

只报告本分支引入且可行动的问题；无证据不报。无问题时明确输出「No findings」。限 400 字。
```

## 3. 记录与闭环

在 `review-log.md` 追加：

```markdown
## Final Review

### Standards - Round <N>
<Standards Reviewer 结果>

### Spec - Round <N>
<Spec Reviewer 结果>

### Disposition - Round <N>
- <finding> → fixed in <commit or non-Git verification evidence> | accepted Minor: <reason> | invalidated as false positive/reclassified: <technical evidence>
```

不在两轴之间挑选「总体最严重」问题。只有证据证明原报告为误报或严重度分类错误时才能作废/重分类；所有已确认的 `Critical / Important` 都必须修复。修复后重跑受影响测试和 Step 6 全量 guard，再对同一审核范围重跑两轴审核。两轴均无阻断项后，Git 项目将 `FINAL_REVIEW_HEAD=$(git rev-parse HEAD)` 记入 `review-log.md`；非 Git 项目记入 `FINAL_REVIEW_MANIFEST`。两者都供 Step 7 检查审核后变更范围。
