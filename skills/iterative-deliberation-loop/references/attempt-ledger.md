# Attempt Ledger

`AttemptLedger` 只保存跨 attempts / branches / sessions 需要恢复的最小 outer 真相。

不要把 transcript 重新塞回 ledger。

## Minimal Structure

```yaml
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
  active_branches: []
  dead_branches: []
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
  - ""
open_questions:
  - ""
attempts:
  - attempt_id:
    branch_label:
    focus_question:
    attempt_kind:
    completion_status:
    strongest_value:
    main_state_delta:
    objection_updates: []
    chosen_action:
    next_focus:
```

### Notes

- `verification_state` 是 controller state，不是 reviewer 建议语气
- `main_state_delta` 只写真正改变下一步控制决策的内容
- `attempts` 是恢复和审计索引，不是第二份 rich history

## Campaign Packet

仅在 handoff / respawn / pause / cross-session 时强制输出。

```text
【主持】Campaign Packet
- campaign_frame:
  - campaign_id：...
  - goal：...
  - desired_artifact：...
  - constraints：...
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
- latest_decision_delta：...
- latest_attempt_summary：...
```

## Attempt Summary

每个 attempt 在 ledger 中只保留压缩摘要：

```text
【主持】Attempt Summary
- attempt_id：...
- branch_label：...
- focus_question：...
- attempt_kind：full_deliberation / exploratory_snapshot
- completion_status：completed / degraded / blocked
- strongest_value：...
- main_state_delta：...
- objection_updates：...
- chosen_action：...
- next_focus：...
```

## Resume Rule

恢复时：

1. 先读 `Campaign Packet`
2. 再决定继续哪条 branch、是否 respawn、是否 stop
3. 再构造一个新的 `AttemptPacket`
4. 再调用 inner engine

不要再维护独立的 `Attempt Resume Packet` 协议。

resume 的目的不是恢复另一套 inner 私有状态机，而是重新发起一次新的合法 attempt。
