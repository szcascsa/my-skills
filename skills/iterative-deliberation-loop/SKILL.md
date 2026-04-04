---
name: iterative-deliberation-loop
description: Use when Codex needs repeated discussion-review-decision cycles on one complex question, must carry forward branch lineage and decisive objections across attempts or sessions, and must decide whether to continue, revise, branch, respawn, or stop.
---

# Iterative Deliberation Loop

把复杂问题组织成一个 `campaign`，而不是一次性圆桌。

这个 skill 只做 outer control：
- 固定跨 attempt 的最小 truth anchors
- 调用 [subagent-discussion](../subagent-discussion/SKILL.md) 跑单次深度讨论
- 用 state-aware reviewer cell 判断本轮是否真的推进
- 在有限动作集合里做下一步裁决
- 在跨 session 时输出可恢复的 packet，而不是要求回放长 transcript

不要把它当成 [subagent-discussion](../subagent-discussion/SKILL.md) 的替代品：
- `subagent-discussion`：单次 attempt 的 inner discussion engine
- `iterative-deliberation-loop`：跨 attempt 的 controller、review gate 和 branch manager

## 适用边界

优先在以下情况使用：
- 科研创新想法需要多轮提出、压测、修正和换方向
- 架构设计存在明显不确定性，需要 reviewer 驱动的连续迭代
- 用户明确要求“根据上一轮意见继续讨论”
- 需要保留 branch lineage、candidate lineage 和 objection state
- 需要跨 session 延续，而不想每次重讲全文

以下情况不要优先使用：
- 只需要一次 roundtable、debate、brainstorm 或多角色评审
- 只是查事实或拿单个答案
- 问题很小，不值得维护 outer state

## 默认值

| 项目 | 默认值 |
| --- | --- |
| outer controller | `intent_anchored_minimal_controller` |
| inner engine | [subagent-discussion](../subagent-discussion/SKILL.md) |
| rubric policy | `semi_frozen` |
| active branch limit | `2` |
| reviewer cell size | `low=1` `medium=2` `high=2-3` |
| attempt budget | `low=2` `medium=3` `high=4-5` |
| heavy modules | `pressure_activated` |
| packet policy | 只在 `cross_session / respawn / handoff / explicit_pause` 时强制输出 |
| close policy | 由 `DecisionDelta` 决定，不再默认一直保留旧 subagents |

如果用户没有给复杂度：
- 科研创新、架构设计默认按 `medium`
- 高风险、高不确定或跨 session 任务默认按 `high`

## Core Design

外层不要维护一整套“对象齐全型 orchestrator”。默认只维护最小 truth anchors。

### Always-On Truth Anchors

| 对象 | 作用 | 最小字段 |
| --- | --- | --- |
| `CampaignFrame` | 定义整个 campaign 的稳定边界 | `campaign_id` `goal` `desired_artifact` `constraints` `effort_budget` `risk_tolerance` `out_of_scope` `stop_condition` `review_rubric` |
| `AttemptIntent` | 定义本次 attempt 想验证什么 | `attempt_id` `parent_attempt_id` `branch_label` `objective_mode` `focus_question` `intended_delta` `success_condition` `roster_strategy` `review_focus` |
| `CandidateLineage` | 定义跨 attempts / branches 的候选谱系 | `candidate_id` `parent_candidate_id` `current_branch` `latest_thesis` |
| `DecisiveObjectionQueue` | 只维护会改变走向的 objection family | `objection_id` `target_candidate_id` `status` `introduced_by_attempt` `evidence_needed` |
| `DecisionDelta` | 记录主持人这轮真正改了什么状态 | `chosen_action` `reason` `state_changes` `roster_change` `branch_change` `next_focus` `verification_state` |
| `BranchStatus` | 记录存活 branch 与关闭理由 | `active_branches` `dead_branches` |
| `AttemptLedger` | 汇总当前 campaign 的最小可恢复状态 | `campaign_frame` `branch_status` `candidate_lineage` `decisive_objections` `verification_state` `stable_findings` `open_questions` `attempts` |

### Pressure-Activated Modules

默认不要常开重型治理层。只在压力出现时激活：

- `ExpandedReviewCell`
  触发：高风险、review 分歧大、连续两轮低增量、或需要 stop / branch / respawn 裁决
- `VerificationGate`
  触发：补某个证据会改变下一控制决策，而不是只会让论证更完整
- `BranchGate`
  触发：任何动作想改 `thesis / framing / success_condition`，都必须解释为什么不是开新 branch
- `RubricChallengeWindow`
  触发：问题定义、最终 artifact 类型或 branch thesis 发生变化
- `FullCampaignPacket / AuditTrailPlus`
  触发：跨 session、多人 handoff、branch 数上升、或用户明确要求完整留痕

默认不常驻：
- transcript 级完整历史
- 全量 open question 编号
- full scorecard
- 默认双 reviewer
- 默认完整 fact ledger

事实核验仍然重要，但默认只维护“会改变下一控制决策”的 verification backlog。

## Compatibility Rule

外层 controller 不要在 inner engine 外面再平行造一套近似对象。

如果 [subagent-discussion](../subagent-discussion/SKILL.md) 返回了这些对象，优先直接复用：
- `ObjectiveSpec`
- `FactSheet`
- `CandidateArtifact`
- `CritiqueMemo`
- `SelectionBoard`
- `ModeDisclosure`
- `ModeratorSummary`

外层只新增 inner engine 没有覆盖的最小 state：
- `AttemptIntent`
- `CandidateLineage`
- `DecisiveObjectionQueue`
- `DecisionDelta`
- `BranchStatus`

旧对象如何收敛到新协议，见 [schema-collapse.md](references/schema-collapse.md)。
外层状态如何映射到 inner contract，见 [controller-inner-mapping.md](references/controller-inner-mapping.md)。

## Core Rule

不要把 reviewer 的意见直接当最终答案。

review 的职责是判断：
- 这轮相对 `AttemptIntent` 有没有真实推进
- 哪个 objection 仍然 load-bearing
- 哪一步最值得继续、分支、换阵容或停止

主持人的职责是：
- 依据 `ReviewVerdict + current state` 做动作选择
- 写清 `state_changes`
- 维护 branch 和 objection 的可恢复真相

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

`AttemptIntent` 是 reviewer 的主基线。没有它，不要让 reviewer 判 `structural_progress`。

`roster_strategy` 优先从有限集合里选：
- `same_core`
- `same_core_add_specialist`
- `swap_generator_cell`
- `swap_critic_cell`
- `fresh_roster`
- `split_branches`

### 4. Run The Inner Attempt

调用 [subagent-discussion](../subagent-discussion/SKILL.md) 执行一次深度讨论。

至少投给 inner engine：
- `topic`
- `objective_mode`
- `final_artifact`
- `attempt_id`
- `focus_question`
- `review_focus`
- `carry_forward`
- `open_decisive_objections`
- `roster_strategy`

如果当前 attempt 来源于父 branch，必须显式写：
- 本次继承了什么
- 本次故意改了什么
- 本次希望关闭或验证哪个 objection
- 本次什么变化算成功，什么变化意味着应该开 branch

### 5. Review The Attempt

reviewer cell 默认与 proposer cell 分离。

reviewer 的输入至少包括：
- `AttemptIntent`
- `baseline_reference`
- 当前 `branch snapshot`
- 当前 `DecisiveObjectionQueue` 摘要
- 最近一次 `DecisionDelta`
- 当前 rubric

reviewer 必须回答：
- `structural_progress`：`none / local / branch_shaping`
- 本轮最强价值是什么
- 哪个 objection 被关闭、只是 address，还是被升级
- 是否出现 branch-worthy 的 thesis drift
- 是否存在必须先过 `VerificationGate` 的证据缺口

review 模板见 [reviewer-rubrics.md](references/reviewer-rubrics.md)。

### 6. Decide The Next Action

主持人只能从有限动作集合里选，见 [branch-decisions.md](references/branch-decisions.md)：
- `continue_same_roster`
- `revise_same_roster`
- `retire_and_respawn`
- `branch_new_direction`
- `stop_and_summarize`

动作选择必须同时满足两类约束：
- [action-state-transitions.md](references/action-state-transitions.md) 定义的字段边界
- [objection-lifecycle.md](references/objection-lifecycle.md) 定义的 objection disposition 规则

不要输出模糊结论，例如“再看看”“再聊一轮”。

必须给出：
- `chosen_action`
- `reason`
- `state_changes`
- `roster_change`
- `branch_change`
- `next_focus`
- `verification_state`

### 7. Update The Ledger

每轮结束只记录真正的 state delta：
- 候选谱系变了什么
- 哪个 decisive objection 被 `introduce / promote / merge / close / reopen / accepted_risk`
- 哪个 branch 存活、关闭或新开
- 下一轮主问题是什么

不要把长 transcript 重新塞回 ledger。
下一轮从 `AttemptLedger` 和 packet 出发，而不是从聊天记录里重新捞重点。

## Hard Gates

以下 gate 默认启用：

1. **Intent Gate**
   没有 `AttemptIntent`，不要发起正式 review

2. **Branch Gate**
   任何动作只要实质改变 `thesis / framing / success_condition`，都必须说明 `why not branch`

3. **Verification Gate**
   如果补某个证据会改变下一控制决策，就把 `verification_state` 设为 `blocked`，先补证据，再继续讨论

4. **Stagnation Gate**
   同一 objection family 在同一 branch 上连续 `2` 次 attempt 没有真实 state delta 时，不要继续装成“只是 revise 一下”

## Anti-Stagnation Rules

默认开启以下规则：

- `structural_progress = none` 且未关闭高优先级 objection 时，禁止 `continue_same_roster`
- 同一类 decision 连续使用超过 `2` 次前，必须说明结构性变化在哪里
- 同一 objection family 连续 `2` 次只是 `addressed` 没有 `closed / accepted_risk / branch_new_direction`，优先 `retire_and_respawn`、`branch_new_direction` 或 `stop_and_summarize`
- 活跃 branch 默认不超过 `2` 条；超过时必须主动 kill value 低的分支
- reviewer 不得只说“更好了”，必须写 `what_would_change_my_mind`
- attempt artifact 不得只交会话摘要，必须交可供审阅的对象
- 如果 review 发现只是换说法没换结构，直接记为 `no structural progress`

## Preset Routing

优先使用 [review-loop.md](references/review-loop.md) 中的 preset sequence：
- 科研创新：先 `idea_funnel`，再 `pressure_test`，必要时开 branch 或收束成实验建议
- 架构设计：先 `architecture_synthesis`，再 `pressure_test`，再对 surviving option 修订
- 复杂模糊问题：先 `sensemaking`，再切到 `idea_funnel` 或 `architecture_synthesis`

不要在 outer controller 里复制 inner engine 的 phase 细节；outer 只决定下一次该用哪个 mode。

## 输出协议

默认面向用户输出以下块：

```text
【主持】Campaign Setup
- topic：...
- goal：...
- desired_artifact：...
- complexity：...
- review_rubric：...
- stop_condition：...

【主持】Ledger Snapshot
- current_branch：...
- active_branches：...
- stable_findings：...
- open_decisive_objections：...
- open_questions：...

【主持】Mode Disclosure
- mode：persistent / replay / spokesperson / degraded_synthesis
- content_origin：participant-originated / moderator-synthesized / mixed
- degrade_event：...
- synthesis_used：yes / no

【主持】Board Snapshot
- survivors：...
- rejected：...
- outlier_kept：...

【Review Cell】Latest Verdict
- judgment：...
- structural_progress：...
- strongest_value：...
- decisive_objections：...
- objection_disposition：...
- branch_opportunity：...
- verification_needed：...
- what_would_change_my_mind：...
- suggested_action：...

【主持】Decision
- chosen_action：...
- reason：...
- state_changes：...
- roster_change：...
- branch_change：...
- next_focus：...
- verification_state：clear / blocked / satisfied

【主持】Next Attempt / Final Recommendation
- ...
```

如果结束，必须给：
- 当前最佳方案或 shortlist
- 为什么不是其他 branch
- 仍未关闭的关键 objection
- 下一步验证建议

## Cross-Session Rule

只有在以下情况强制输出 `Campaign Packet`：
- campaign 将跨 session 继续
- 需要 handoff 给新主持人或新 roster
- 即将 `retire_and_respawn`
- 用户明确暂停，稍后继续

跨 session 时默认输出两层 packet：
- `Campaign Packet`：outer canonical snapshot，恢复 `CampaignFrame / BranchStatus / CandidateLineage / DecisiveObjectionQueue / DecisionDelta / verification_state`
- `Attempt Resume Packet`：inner authoritative resume contract，恢复本轮所需的 `AttemptIntent` 关键信息、`ObjectiveSpec / FactSheet / SelectionBoard / ModeDisclosure` 摘要，以及 routing 信息

packet 模板见 [attempt-ledger.md](references/attempt-ledger.md)。
worked example 见 [worked-example.md](references/worked-example.md)。

新会话恢复时：
- 先读 packet
- 再决定继续同一 branch、开新 branch 还是停止
- 不要要求完整复述全部历史

## 实施提示

按需读取：
- [schema-collapse.md](references/schema-collapse.md)
- [controller-inner-mapping.md](references/controller-inner-mapping.md)
- [review-loop.md](references/review-loop.md)
- [reviewer-rubrics.md](references/reviewer-rubrics.md)
- [branch-decisions.md](references/branch-decisions.md)
- [action-state-transitions.md](references/action-state-transitions.md)
- [objection-lifecycle.md](references/objection-lifecycle.md)
- [attempt-ledger.md](references/attempt-ledger.md)
- [worked-example.md](references/worked-example.md)
- [adversarial-scenarios.md](references/adversarial-scenarios.md)
