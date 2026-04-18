# Controller-to-Inner Contract Mapping

这个文件定义 outer controller 如何通过 boundary contract 调用 inner `subagent-discussion`。

目标：

- 明确哪个字段 authoritative
- 明确 outer state 如何收敛到 `AttemptPacket`
- 明确 `AttemptResult` 如何被提升成 outer ledger state
- 避免 outer 和 inner 各自维护半套真相

现行协议见：

- [lifecycle-ownership.md](lifecycle-ownership.md)
- [attempt-boundary.md](attempt-boundary.md)

## Mapping Table: Outer -> `AttemptPacket`

| Outer state | Packet field | authoritative owner | 说明 |
| --- | --- | --- | --- |
| `CampaignFrame.goal` | none | outer | inner 不直接消费 campaign goal；必要信息先压进 `focus_question` 或 `carry_forward_summary` |
| `CampaignFrame.desired_artifact` | `final_artifact` | outer | inner 不应静默改 artifact 类型 |
| `CampaignFrame.constraints` | `out_of_scope` / `carry_forward_summary` | outer | 只传本次 attempt 真正相关的约束 |
| `AttemptIntent.attempt_id` | `attempt_id` | outer | 稳定 attempt identity |
| `AttemptIntent.branch_label` | `branch_label` | outer | inner 只消费，不改写 |
| `AttemptIntent.objective_mode` | `objective_mode` | outer | inner 按 mode 执行 |
| `AttemptIntent.focus_question` | `focus_question` | outer | 本次 attempt 唯一主问题 |
| `AttemptIntent.success_condition` | `success_condition` | outer | reviewer 后续用它判断是否推进 |
| `AttemptIntent.review_focus` | `review_focus` | outer | inner 需要知道本次结果将如何被审 |
| `AttemptIntent.roster_strategy` | `roster_policy.strategy` | outer | inner 只消费本次 roster 策略 |
| `DecisiveObjectionQueue` subset | `decisive_objections_in_scope` | outer | 只传本次真正需要回应的 objections |
| previous state summary | `carry_forward_summary` | outer | 压缩前情，不直接回灌 transcript |
| runtime preference | `runtime_request` | outer | outer 决定希望的 guarantee / mode / round budget |

## Mapping Table: `AttemptResult` -> Outer State

| `AttemptResult` field | Outer destination | authoritative owner | 说明 |
| --- | --- | --- | --- |
| `objective_spec_effective` | reviewer input / audit trail | inner | 只记录，不回抄成新 outer object |
| `fact_sheet_effective` | reviewer input / audit trail | inner | outer 可引用，不重建第二份事实账本 |
| `board_snapshot` | reviewer input + candidate promotion | inner | 用户可见 shortlist 直接复用 |
| `candidate_updates` | `CandidateLineage` | outer | inner 产出 update，outer 决定跨 attempt lineage |
| `objection_updates` | `DecisiveObjectionQueue` | outer | inner 产出 update，outer 只提升 route-changing family |
| `artifact` | reviewer input | inner | reviewer 审这个对象，不审 transcript |
| `mode_disclosure` | user-visible transparency | inner | outer 默认透传，不重新发明 mode 词汇 |
| `runtime_status` | `DecisionDelta.roster_change` input | inner recommendation -> outer decision | inner 只能建议，outer 才能决定下一 attempt roster |
| `next_decision_inputs` | reviewer + controller input | inner | 帮 outer 快速理解 strongest value / evidence gap |

## Adapter Rules

### 1. Only Two Boundary Objects

跨层只允许：

- `AttemptPacket`
- `AttemptResult`

如果需要第三个对象，优先问自己：

- 能否并入 packet
- 能否并入 result
- 还是这个状态根本不该跨层

### 2. inner 没返回，不要伪造 rich object

如果 inner 没返回 `board_snapshot` 或 `mode_disclosure`：

- outer 可以明确写缺失
- 不要假装这些对象已经存在

### 3. outer 只提升 route-changing 内容

`objection_updates` 里的所有问题，不会自动进入 `DecisiveObjectionQueue`。

只有会改变下一控制决策的 objection 才值得提升。

### 4. lineage 由 outer 统一管理

inner 的 `candidate_id` 是本轮产物。

跨 branch、跨 attempt 的 lineage 归 outer 管。

### 5. replay 不是另一套协议

cross-session 或 respawn 时：

- 先恢复 `Campaign Packet`
- 再重新构造一个新的 `AttemptPacket`

不要再维护独立的 `Attempt Resume Packet` 协议。

### 6. named targets 只在 inner round routing 出现

`named_targets` 属于 inner round routing，不属于 outer always-on anchor。

它应存在于 inner round packet，而不是 outer boundary object。
