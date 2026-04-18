# Worked Example

这个例子演示一个 `3-attempt` campaign 如何围绕 `AttemptPacket / AttemptResult` 推进，而不是每轮重建全量协议。

## Topic

如何继续改进 `subagent-discussion`

## Campaign Setup

```text
goal: 找到最高杠杆的下一版 skill 设计
desired_artifact: prioritized improvement slate
review_rubric: architecture_design
stop_condition: 找到一个明显优于其他方向的可执行主方案
```

## Attempt 1

### AttemptIntent

```text
attempt_id: A1
branch_label: main
objective_mode: architecture_synthesis
focus_question: 现有 skill 最缺哪类能力
intended_delta: 形成 2-3 个可区分升级方向
success_condition: 产出可比较的 shortlist
```

### AttemptPacket

```text
attempt_id: A1
branch_label: main
topic: 如何继续改进 `subagent-discussion`
objective_mode: architecture_synthesis
final_artifact: prioritized improvement slate
focus_question: 现有 skill 最缺哪类能力
success_condition: 产出可比较的 shortlist
review_focus:
- shortlist quality
- distinction between directions
carry_forward_summary: []
decisive_objections_in_scope: []
runtime_request:
  attempt_kind: full_deliberation
  runtime_mode: persistent
```

### AttemptResult

```text
board_snapshot:
- survivors:
  - `heavy-governance`
  - `lazy-state`
  - `review-hardening`
completion_status: completed
```

### ReviewVerdict

```text
judgment: salvageable
structural_progress: branch_shaping
decisive_objections:
- O1: `heavy-governance` 太重
- O2: `lazy-state` 把 truth anchors 推迟得太晚
- O3: `review-hardening` 不能单独充当 controller
suggested_action: branch_new_direction
```

### DecisionDelta

```text
chosen_action: branch_new_direction
state_changes:
- create branch `minimal-controller`
- keep O1 O2 O3 open
- mark original three directions as feeder ideas, not active end states
next_focus: 把三者压成一个更轻的分层控制模型
```

## Attempt 2

### AttemptIntent

```text
attempt_id: A2
parent_attempt_id: A1
branch_label: minimal-controller
objective_mode: architecture_synthesis
focus_question: 最小 truth anchors 应该是什么
intended_delta: 把常驻状态压缩到能恢复和裁决所需的最小集合
success_condition: 提出比旧协议更轻且不丢真相锚点的 outer contract
```

### AttemptPacket

```text
attempt_id: A2
branch_label: minimal-controller
objective_mode: architecture_synthesis
focus_question: 最小 truth anchors 应该是什么
review_focus:
- outer state minimality
- handoff safety
carry_forward_summary:
- A1 shortlisted three upgrade directions
- O1 O2 O3 remain open
decisive_objections_in_scope:
- O1
- O2
- O3
runtime_request:
  attempt_kind: full_deliberation
  runtime_mode: persistent
```

### AttemptResult

```text
artifact:
- `CampaignFrame`
- `CandidateLineage`
- `DecisiveObjectionQueue`
- `DecisionDelta`
- `BranchStatus`
board_snapshot:
- survivors:
  - `minimal-controller`
objection_updates:
- O1 mitigated
- O2 mitigated
completion_status: completed
```

### ReviewVerdict

```text
judgment: salvageable
structural_progress: local
decisive_objections:
- O1: `太重` 已显著缓解
- O2: `truth anchors 不能 lazy` 已显著缓解
- O4: 同 branch 动作仍可偷偷换 thesis
- O5: objection lifecycle 还不够明确
objection_disposition: O1 mitigated, O2 mitigated, O4 open, O5 open
suggested_action: revise_same_roster
```

### DecisionDelta

```text
chosen_action: revise_same_roster
state_changes:
- mark O1 `closed`
- mark O2 `closed`
- introduce O4 `same-branch semantic drift`
- introduce O5 `silent objection loss`
next_focus: 补 Attempt boundary、BranchGate 和 objection lifecycle
```

## Attempt 3

### AttemptIntent

```text
attempt_id: A3
parent_attempt_id: A2
branch_label: minimal-controller
objective_mode: pressure_test
focus_question: 如何阻止 `revise_same_roster` 偷带 branch drift
intended_delta: 明确 action-state boundaries 与 objection lifecycle
success_condition: reviewer 能区分 local repair 和 hidden branch
```

### AttemptPacket

```text
attempt_id: A3
branch_label: minimal-controller
objective_mode: pressure_test
focus_question: 如何阻止 `revise_same_roster` 偷带 branch drift
review_focus:
- boundary strictness
- objection lifecycle completeness
carry_forward_summary:
- A2 established minimal outer anchors
- O4 and O5 remain open
decisive_objections_in_scope:
- O4
- O5
runtime_request:
  attempt_kind: full_deliberation
  runtime_mode: replay
```

### AttemptResult

```text
artifact:
- add `AttemptPacket`
- add `AttemptResult`
- add `BranchGate`
- unify objection status set
objection_updates:
- O4 closed
- O5 closed
completion_status: completed
mode_disclosure:
  mode: replay
```

### ReviewVerdict

```text
judgment: promising
structural_progress: local
decisive_objections:
- O4 closed
- O5 closed
objection_disposition: closed
suggested_action: stop_and_summarize
```

### DecisionDelta

```text
chosen_action: stop_and_summarize
state_changes:
- close branch `minimal-controller` as winner
- archive feeder ideas
- output `Campaign Packet` for implementation
next_focus: 落 skill patch
```

## 为什么这个例子重要

这个例子同时演示了：

- branch 是因为 thesis family 变化，而不是因为 review 风格变化
- `mitigated` 不等于 `closed`
- packet 只在 handoff / stop 边界输出
- reviewer 以 `AttemptIntent + AttemptResult` 为基线，而不是只看论述有没有更顺
