# Attempt Boundary

这个文件定义 outer controller 与 inner discussion engine 之间唯一合法的交换面。

不要再靠 prose mapping、主持人口头摘要或隐式字段注入来传递状态。

## Core Rule

跨层只允许两种顶层对象：

- `AttemptPacket`
- `AttemptResult`

如果某个字段不在这两个对象里，它就不是 authoritative boundary state。

## Design Goals

- outer 能稳定发起一次 attempt，而不是把 controller 私有字段塞进 inner
- inner 能稳定返回一次可审、可恢复的结果，而不是让 outer 从自由文本里猜状态
- replay / respawn / pause / cross-session 都基于同一份 conserved attempt state
- `full_deliberation` 与 `exploratory_snapshot` 的保证级别显式区分

## Authoritative Enums

### `attempt_kind`

- `full_deliberation`
- `exploratory_snapshot`

### `runtime_mode`

- `persistent`
- `replay`
- `degraded`

### `completion_status`

- `completed`
- `degraded`
- `blocked`

### objection `status`

- `open`
- `addressed`
- `mitigated`
- `closed`
- `accepted_risk`
- `reopened`
- `escalated`

## `AttemptPacket`

outer 发起 inner attempt 时，只能发送这一对象。

```yaml
AttemptPacket:
  campaign_id: ""
  attempt_id: ""
  branch_label: ""
  topic: ""
  objective_mode: ""
  final_artifact: ""
  focus_question: ""
  success_condition: ""
  review_focus: []
  carry_forward_summary: []
  decisive_objections_in_scope: []
  roster_policy:
    strategy: same_core | same_core_add_specialist | swap_generator_cell | swap_critic_cell | fresh_roster | split_branches
    participant_roster: []
  fact_sheet_ref: {}
  runtime_request:
    attempt_kind: full_deliberation | exploratory_snapshot
    runtime_mode: persistent | replay | degraded
    round_budget: 0
  out_of_scope: []
```

### Field Rules

| 字段 | authoritative owner | consumed by | 说明 |
| --- | --- | --- | --- |
| `campaign_id` | outer | boundary / outer ledger | 只用于跨 attempt 追踪 |
| `attempt_id` | outer | inner / outer | 本次 attempt 的稳定 identity |
| `branch_label` | outer | inner / outer | inner 不得擅自改写 |
| `topic` | outer | inner | 用户问题的稳定主题 |
| `objective_mode` | outer | inner | inner 按 mode 执行 |
| `final_artifact` | outer | inner | inner 不得静默切 artifact 类型 |
| `focus_question` | outer | inner | 本次 attempt 唯一主问题 |
| `success_condition` | outer | inner / reviewer | 用于 review 本次是否推进 |
| `review_focus` | outer | inner / reviewer | 告诉 inner 本次结果会被怎样审 |
| `carry_forward_summary` | outer | inner | 压缩前情，不直接回灌 transcript |
| `decisive_objections_in_scope` | outer | inner | 只传 route-changing objections |
| `roster_policy` | outer | inner | inner 消费本次 roster 策略，不拥有跨 attempt roster 决策 |
| `fact_sheet_ref` | outer | inner | 若 outer 已核实，可直接传入；否则由 inner 生成最小 `FactSheet` |
| `runtime_request` | outer | inner | outer 指定希望的 guarantee 和运行方式 |
| `out_of_scope` | outer | inner | attempt 明确不讨论什么 |

## `AttemptResult`

inner 完成一次 attempt 后，只能返回这一对象。

```yaml
AttemptResult:
  campaign_id: ""
  attempt_id: ""
  branch_label: ""
  attempt_kind: full_deliberation | exploratory_snapshot
  completion_status: completed | degraded | blocked
  mode_disclosure:
    mode: persistent | replay | degraded
    content_origin: participant-originated | moderator-synthesized | mixed
    degrade_event: ""
    synthesis_used: yes | no
  objective_spec_effective: {}
  fact_sheet_effective: {}
  board_snapshot:
    survivors: []
    rejected: []
    outlier_kept: []
  candidate_updates: []
  objection_updates: []
  artifact: {}
  runtime_status:
    participant_liveness: retain | release | replace
    replay_required_next_time: true | false
  next_decision_inputs:
    strongest_value: ""
    open_objection_hints: []
    evidence_gaps: []
```

### Field Rules

| 字段 | authoritative owner | consumed by | 说明 |
| --- | --- | --- | --- |
| `attempt_kind` | inner, constrained by packet | outer / reviewer | 明确这次结果属于 full 还是 snapshot |
| `completion_status` | inner | outer | 告诉 outer 是否 completed / degraded / blocked |
| `mode_disclosure` | inner | outer / user | user-facing transparency；不得再套另一份 mode 词汇 |
| `objective_spec_effective` | inner | outer / reviewer | 本次真正执行的 attempt spec |
| `fact_sheet_effective` | inner | outer / reviewer | 本次真正消费的事实边界 |
| `board_snapshot` | inner | outer / reviewer | 本次结束时的 survivor / rejected / outlier |
| `candidate_updates` | inner | outer | 本次 attempt 中 candidate 的主要更新 |
| `objection_updates` | inner | outer | 本次 attempt 内部真正产生或处理的 objection 更新 |
| `artifact` | inner | reviewer / user | 本次可审结果 |
| `runtime_status` | inner | outer | 只给 liveness recommendation，不做下一 attempt 决策 |
| `next_decision_inputs` | inner | outer / reviewer | review 和 controller 下一步需要的最小输入 |

## Boundary Invariants

### 1. No Raw Controller Leakage

outer 不得绕过 `AttemptPacket` 直接要求 inner 消费额外字段。

如果某个字段经常被 outer 传入但不在 `AttemptPacket` 中：

- 把它升级进 schema
- 或删除它

不要长期维持“文档里要求、schema 里没有”的灰色地带。

### 2. No Free-Text State Recovery

outer 不得从自由文本里恢复 canonical state。

如果一个状态需要 survive 到下一次 attempt，它必须已经在 `AttemptResult` 里。

### 3. Snapshot Is Not Full Deliberation

`exploratory_snapshot` 只能提供较弱保证：

- 可以生成 candidate draft
- 可以给出风险 hints
- 可以给出 partial board
- 不得伪装成已经完成 full `critique / repair / judge`

### 4. One Mode Vocabulary

user-facing mode 与 controller 消费的 runtime mode 使用同一套 enum：

- `persistent`
- `replay`
- `degraded`

把 `spokesperson`、`degraded_synthesis` 之类旧术语降级为 `degrade_event`，不要再并列成 mode。

### 5. One Objection Vocabulary

所有 objection 更新都使用同一套 status enum。

inner 可以产出：

- `open`
- `addressed`
- `mitigated`
- `closed`

outer 在 controller 层可以追加：

- `accepted_risk`
- `reopened`
- `escalated`

## Upgrade Rules

- `AttemptResult.board_snapshot` 可以直接喂给 reviewer
- `AttemptResult.objection_updates` 经过 outer promotion 后，才进入 `DecisiveObjectionQueue`
- `AttemptResult.runtime_status` 只能推荐 roster 处理方式，不能直接替代 `DecisionDelta`
- `AttemptResult.completion_status = degraded` 时，outer 不得把本次当作 full structural progress 的强证据

## Minimal Example

```yaml
AttemptPacket:
  campaign_id: campaign-7
  attempt_id: A2
  branch_label: main
  topic: "继续压测新的 deliberation contract"
  objective_mode: pressure_test
  final_artifact: decision memo
  focus_question: "当前 packet 是否足够支撑 replay 和 review"
  success_condition: "明确指出缺失字段或确认 packet 足够"
  review_focus:
    - "packet completeness"
    - "resume safety"
  carry_forward_summary:
    - "A1 produced initial packet draft"
  decisive_objections_in_scope:
    - "O4 packet may lose objection identity on replay"
  roster_policy:
    strategy: same_core
    participant_roster: []
  runtime_request:
    attempt_kind: full_deliberation
    runtime_mode: persistent
    round_budget: 3
```

```yaml
AttemptResult:
  campaign_id: campaign-7
  attempt_id: A2
  branch_label: main
  attempt_kind: full_deliberation
  completion_status: completed
  mode_disclosure:
    mode: persistent
    content_origin: participant-originated
    degrade_event: ""
    synthesis_used: no
  board_snapshot:
    survivors:
      - packet-v2
    rejected:
      - packet-v1
    outlier_kept: []
  objection_updates:
    - objection_id: O4
      status: mitigated
      basis: "candidate and objection identity now survive replay"
  runtime_status:
    participant_liveness: retain
    replay_required_next_time: false
  next_decision_inputs:
    strongest_value: "packet-v2 now preserves objection identity"
    open_objection_hints:
      - "O5 runtime mode still under-specified"
    evidence_gaps: []
```
