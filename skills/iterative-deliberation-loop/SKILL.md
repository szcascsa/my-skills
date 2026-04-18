---
name: iterative-deliberation-loop
description: This skill should be used when the user asks to continue a complex discussion across attempts or sessions, carry forward decisive objections and branch history, and decide whether to continue, revise, branch, respawn, or stop.
---

# Iterative Deliberation Loop

把复杂问题组织成一个 `campaign`，而不是一次性圆桌。

这个 skill 只做 outer control：

- 固定跨 attempt 的最小 truth anchors
- 构造一次合法的 `AttemptPacket`
- 调用 [subagent-discussion](../subagent-discussion/SKILL.md) 跑单次 deep attempt
- 审阅 `AttemptResult`
- 在有限动作集合里做下一步裁决
- 在跨 session 时输出可恢复的 `Campaign Packet`

不要把它当成 [subagent-discussion](../subagent-discussion/SKILL.md) 的替代品：

- `subagent-discussion`
  单次 attempt 的 inner discussion engine
- `iterative-deliberation-loop`
  跨 attempt 的 controller、review gate 和 branch manager

## 适用边界

优先在以下情况使用：

- 科研创新想法需要多轮提出、压测、修正和换方向
- 架构设计存在明显不确定性，需要 reviewer 驱动的连续迭代
- 用户明确要求“根据上一轮意见继续讨论”
- 需要保留 branch lineage、candidate lineage 和 decisive objection state
- 需要跨 session 延续，而不想每次重讲全文

以下情况不要优先使用：

- 只需要一次 roundtable、debate、brainstorm 或多角色评审
- 只是查事实或拿单个答案
- 问题很小，不值得维护 outer controller state

## Authoritative References

这两个文件是现行协议，不是补充说明：

- [lifecycle-ownership.md](references/lifecycle-ownership.md)
- [attempt-boundary.md](references/attempt-boundary.md)

如果 top-level prose 与这两个文件冲突，以它们为准。

## Default Shape

| 项目 | 默认值 |
| --- | --- |
| outer controller | `intent_anchored_minimal_controller` |
| inner engine | [subagent-discussion](../subagent-discussion/SKILL.md) |
| rubric policy | `semi_frozen` |
| active branch limit | `2` |
| reviewer cell size | `low=1` `medium=2` `high=2-3` |
| attempt budget | `low=2` `medium=3` `high=4-5` |
| packet policy | pause / handoff / respawn / cross-session 时输出 `Campaign Packet` |

如果用户没有给复杂度：

- 科研创新、架构设计默认按 `medium`
- 高风险、高不确定或跨 session 任务默认按 `high`

## Always-On Outer State

outer 默认只常驻以下对象：

| 对象 | 作用 |
| --- | --- |
| `CampaignFrame` | 定义整个 campaign 的稳定边界 |
| `AttemptIntent` | 定义本次 attempt 想验证什么 |
| `CandidateLineage` | 记录跨 attempt / branch 的 candidate 谱系 |
| `DecisiveObjectionQueue` | 记录会改变下一控制决策的 objection family |
| `DecisionDelta` | 记录 controller 这轮真正改了什么状态 |
| `BranchStatus` | 记录存活 branch 与关闭理由 |
| `AttemptLedger` | 记录最小可恢复 outer state |
| `verification_state` | 只记录会改变下一控制决策的证据阻塞状态 |

outer 不重复维护 inner discussion objects。

## Core Rule

不要把 reviewer 的意见直接当最终答案。

review 的职责是判断：

- 这轮相对 `AttemptIntent` 有没有真实推进
- 哪个 decisive objection 仍然 load-bearing
- 哪一步最值得继续、分支、换阵容或停止

controller 的职责是：

- 基于 `AttemptResult + ReviewVerdict + current outer state` 选动作
- 写清 `DecisionDelta.state_changes`
- 维护 branch、objection 和 ledger 的可恢复真相

## Workflow

### 1. Frame Campaign

锁定：

- `topic`
- `goal`
- `desired_artifact`
- `constraints`
- `effort_budget`
- `risk_tolerance`
- `out_of_scope`
- `complexity`
- `stop_condition`
- `review_rubric`

没有这些字段时，优先自动补齐，不要把 round 0 变成配置地狱。

### 2. Freeze Review Rubric

优先从 [reviewer-rubrics.md](references/reviewer-rubrics.md) 选 preset。

默认 `semi_frozen`：

- 小修、小 branch 内修补时，不改 rubric
- 只有问题定义、artifact 类型或 branch thesis 变了，才允许改 rubric
- 改 rubric 时必须写 `why rubric changed`

### 3. Plan One Attempt

每个 attempt 只追一个主问题。

`AttemptIntent` 至少补齐：

- `attempt_id`
- `parent_attempt_id`
- `branch_label`
- `objective_mode`
- `focus_question`
- `intended_delta`
- `success_condition`
- `roster_strategy`
- `review_focus`

`AttemptIntent` 是 reviewer 的主基线。没有它，不要发起正式 review。

### 4. Build `AttemptPacket`

不要把 controller 私有字段直接塞进 inner。

先按照 [attempt-boundary.md](references/attempt-boundary.md) 构造一个 `AttemptPacket`，只包含：

- attempt identity
- focus question
- success condition
- carry-forward summary
- decisive objections in scope
- roster policy
- runtime request

如果当前 attempt 来源于父 branch，必须在 `carry_forward_summary` 中明确写：

- 继承了什么
- 故意改了什么
- 希望关闭或验证哪个 objection
- 什么变化算成功，什么变化意味着应该开 branch

### 5. Run The Inner Attempt

调用 [subagent-discussion](../subagent-discussion/SKILL.md)，让 inner 只消费 `AttemptPacket`。

inner 只允许返回 `AttemptResult`。

### 6. Review The `AttemptResult`

reviewer cell 默认与 proposer cell 分离。

reviewer 的最小输入至少包括：

- `AttemptIntent`
- 当前 `CampaignFrame`
- 当前 `BranchStatus`
- 当前 `DecisiveObjectionQueue` 摘要
- 最近一次 `DecisionDelta`
- 本轮 `AttemptResult`
- 当前 rubric

reviewer 必须回答：

- `structural_progress`: `none / local / branch_shaping`
- 本轮最强价值是什么
- 哪个 objection 被 `addressed / mitigated / closed / escalated`
- 是否出现 branch-worthy thesis drift
- 是否存在必须先过 `VerificationGate` 的证据缺口

如果 `AttemptResult.attempt_kind = exploratory_snapshot` 或 `completion_status = degraded`，reviewer 必须降低证据权重，不得把它当成 full attempt 的强证据。

### 7. Decide The Next Action

主持人只能从有限动作集合里选，见 [branch-decisions.md](references/branch-decisions.md)：

- `continue_same_roster`
- `revise_same_roster`
- `retire_and_respawn`
- `branch_new_direction`
- `stop_and_summarize`

动作选择必须同时满足：

- [action-state-transitions.md](references/action-state-transitions.md) 的字段边界
- [objection-lifecycle.md](references/objection-lifecycle.md) 的状态转移规则

必须给出：

- `chosen_action`
- `reason`
- `state_changes`
- `roster_change`
- `branch_change`
- `next_focus`
- `verification_state`

### 8. Update The Ledger

每轮结束只记录真正的 outer state delta：

- 候选谱系变了什么
- 哪个 decisive objection 被引入、升级、合并，或状态改写为 `addressed / mitigated / closed / accepted_risk / reopened / escalated`
- 哪个 branch 存活、关闭或新开
- 下一轮主问题是什么

不要把 transcript 重新塞回 ledger。

## Hard Gates

### 1. Intent Gate

没有 `AttemptIntent`，不要发起正式 review。

### 2. Boundary Gate

没有合法 `AttemptPacket`，不要调用 inner。

没有合法 `AttemptResult`，不要更新 outer ledger。

### 3. Branch Gate

任何动作只要实质改变 `thesis / framing / success_condition`，都必须说明 `why not branch`。

### 4. Verification Gate

如果补某个证据会改变下一控制决策，就把 `verification_state.status` 设为 `blocked`，先补证据，再继续讨论。

### 5. Snapshot Gate

`exploratory_snapshot` 不得伪装成 full attempt。

如果本轮没有经过 full `critique / repair / judge` guarantee，outer 不得把它写成“已完成 structural closure”。

## Anti-Stagnation Rules

- `structural_progress = none` 且未关闭高优先级 objection 时，禁止 `continue_same_roster`
- 同一类 decision 连续使用超过 `2` 次前，必须说明结构性变化在哪里
- 同一 objection family 连续 `2` 次只有 `addressed` 而没有 `mitigated / closed / accepted_risk / branch_new_direction` 时，优先 `retire_and_respawn`、`branch_new_direction` 或 `stop_and_summarize`
- 活跃 branch 默认不超过 `2` 条；超过时必须主动 kill value 低的分支
- reviewer 不得只说“更好了”，必须写 `what_would_change_my_mind`

## Cross-Session Rule

跨 session 时只强制输出：

- `Campaign Packet`

不要再维护独立的 `Attempt Resume Packet` 协议。

恢复流程：

1. 读取 `Campaign Packet`
2. 判断继续当前 branch、开新 branch、respawn 还是停止
3. 重新构造一个新的 `AttemptPacket`
4. 再调用 inner engine

也就是说，resume 不是恢复另一套 inner 私有协议，而是重新发起一次新的合法 attempt。

## 输出协议

默认面向用户输出以下块：

```text
【主持】Campaign Setup
- topic：...
- goal：...
- desired_artifact：...
- stop_condition：...

【主持】Ledger Snapshot
- current_branch：...
- active_branches：...
- stable_findings：...
- open_decisive_objections：...
- verification_state：clear / blocked / satisfied

【Review Cell】Latest Verdict
- judgment：...
- structural_progress：...
- strongest_value：...
- objection_disposition：open / addressed / mitigated / closed / escalated / accepted_risk
- verification_needed：...
- suggested_action：...

【主持】Decision
- chosen_action：...
- reason：...
- state_changes：...
- roster_change：...
- branch_change：...
- next_focus：...
```

如果结束，必须给：

- 当前最佳方案或 shortlist
- 为什么不是其他 branch
- 仍未关闭的关键 objection
- 下一步验证建议

## 实施提示

按需读取：

- [lifecycle-ownership.md](references/lifecycle-ownership.md)
- [attempt-boundary.md](references/attempt-boundary.md)
- [controller-inner-mapping.md](references/controller-inner-mapping.md)
- [review-loop.md](references/review-loop.md)
- [reviewer-rubrics.md](references/reviewer-rubrics.md)
- [branch-decisions.md](references/branch-decisions.md)
- [action-state-transitions.md](references/action-state-transitions.md)
- [objection-lifecycle.md](references/objection-lifecycle.md)
- [attempt-ledger.md](references/attempt-ledger.md)
- [worked-example.md](references/worked-example.md)
- [schema-collapse.md](references/schema-collapse.md)
