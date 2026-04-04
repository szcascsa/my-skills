# Attempt Ledger

`AttemptLedger` 是这个 skill 的记忆中枢。

没有 ledger，多轮讨论很容易退化成：
- 重复上轮内容
- branch lineage 漂移
- decisive objection 静默丢失
- 无法解释为什么继续、为什么分支、为什么停止

## Ledger Schema

```text
campaign_frame:
  campaign_id:
  goal:
  desired_artifact:
  constraints:
  effort_budget:
  risk_tolerance:
  out_of_scope:
  stop_condition:
  review_rubric:
branch_status:
  active_branches:
  dead_branches:
candidate_lineage:
  - candidate_id:
    parent_candidate_id:
    current_branch:
    latest_thesis:
decisive_objections:
  - objection_id:
    target_candidate_id:
    status:
    introduced_by_attempt:
    evidence_needed:
verification_state:
  status:
  pending_claims:
  decision_blocked:
  verification_result:
stable_findings:
open_questions:
attempts:
  - attempt_id:
    parent_attempt_id:
    branch_label:
    focus_question:
    intended_delta:
    success_condition:
    artifact_summary:
    board_snapshot:
    mode_disclosure:
    strongest_value:
    structural_progress:
    objection_updates:
    review_judgment:
    chosen_action:
    state_changes:
    roster_change:
    branch_change:
    verification_state:
    next_focus:
```

## 记录原则

- `artifact_summary` 只写真正的内容增量
- `board_snapshot` 和 `mode_disclosure` 只保留最近一轮恢复 inner runtime 必需的最小摘要
- `strongest_value` 说明这次 attempt 为什么没白做
- `structural_progress` 只允许 `none / local / branch_shaping`
- `objection_updates` 只记录会改变走向的 objection family
- `chosen_action` 必须是有限动作集合之一
- `verification_state` 是 controller state，不是 reviewer 建议语气
- `state_changes` 必须说明 candidate / branch / objection 到底变了什么
- `candidate_lineage` 维护跨 branch 的 candidate family，而不是 verbose scorecard
- `AttemptIntent` 是 review 基线；没有 `AttemptIntent`，不要写正式 `ReviewVerdict`

## Campaign Packet

跨 session、handoff 或 `retire_and_respawn` 前，至少输出这个 packet：

```text
【主持】Campaign Packet
- campaign_frame：
  - campaign_id：...
  - goal：...
  - desired_artifact：...
  - constraints：...
  - effort_budget：...
  - risk_tolerance：...
  - out_of_scope：...
  - stop_condition：...
  - review_rubric：...
- branch_status：
  - active_branches：...
  - dead_branches：...
- candidate_lineage：...
- decisive_objections：...
- verification_state：
  - status：clear / blocked / satisfied
  - pending_claims：...
  - decision_blocked：...
  - verification_result：...
- stable_findings：...
- open_questions：...
- last_decision_delta：...
- recommended_next_action：...
```

## Attempt Resume Packet

如果下一轮还要继续调用 inner engine，再补一个面向 `subagent-discussion` 的 resume packet：

```text
【主持】Attempt Resume Packet
- attempt_intent：
  - attempt_id：...
  - parent_attempt_id：...
  - branch_label：...
  - objective_mode：...
  - focus_question：...
  - intended_delta：...
  - success_condition：...
  - roster_strategy：...
  - review_focus：...
- objective_spec：goal / objective_mode / final_artifact / success_criteria / stop_condition
- fact_status：...
- carry_forward：...
- open_decisive_objections：...
- candidate_lineage_in_scope：...
- current_board：survivors / rejected / outlier_kept
- last_mode_disclosure：...
- what_changed_since_last_round：...
- named_targets：...
- verification_state：clear / blocked / satisfied
```

`Campaign Packet` 负责恢复 outer state。
`Attempt Resume Packet` 负责恢复 inner runtime。

## Branch 规则

- `parent_attempt_id` 为空表示主线起点
- 开 branch 时必须写清 `why not revise`
- branch 被关闭时，把关闭理由记入 `dead_branches`
- 同一时间默认最多保留 `2` 条活跃 branch
- 同一 branch 上如果 `thesis / framing / success_condition` 实质变化，就不是普通 revise

## 何时可以只更新 Packet 而不重放全文

满足以下条件即可只依赖 packet：
- 上一轮 artifact 已结构化
- reviewer verdict 已结构化
- branch lineage 清楚
- decisive objections 没有丢
- 下一轮只需要知道上轮结论，而不是逐字复盘对话
