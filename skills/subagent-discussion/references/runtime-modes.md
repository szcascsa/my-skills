# Runtime Modes

这个文件只描述 inner discussion engine 的 runtime / orchestration 语义，不重复 boundary 协议。

## 1. Mode Set

只使用一套 runtime mode：

- `persistent`
- `replay`
- `degraded`

### `persistent`

优先模式。

适用条件：

- 当前环境支持创建常驻 subagent
- 主持人可以在多轮中继续向同一 agent 投递

要求：

- 保留 `participant_id -> agent_id` 映射
- 每轮记录最近一次已知立场、未回应问题与当前 phase role
- attempt 内维护 candidate context，避免后续失去上下文

### `replay`

第一层降级。

适用条件：

- 当前环境不能稳定维持常驻 agent
- 或常驻 agent 已失活，需要重建

要求：

- 每轮重发最小前情包
- 至少带上：
  - `objective_spec`
  - `quality_rubric`
  - `verified_facts`
  - `your_last_position`
  - `what_changed_since_last_round`
  - `named_targets`
  - `points_you_must_address`

### `degraded`

最后一层降级。

适用条件：

- 无法维持完整多轮
- 或当前轮已有足够信息，继续细分收益很低

要求：

- 主持人显式告知当前已降级
- 结果必须写进 `AttemptResult.completion_status = degraded`
- outer 不得把本次结果当成 full deliberation 的强证据

## 2. Attempt Kinds

inner 支持两种 attempt kind：

- `full_deliberation`
- `exploratory_snapshot`

### `full_deliberation`

默认模式。

要求：

- phase pipeline 至少覆盖 `frame -> blind_diverge -> critique -> judge`
- 若交付 recommendation，还应覆盖 `repair` 或显式声明无需 repair 的理由
- 结束前输出 `SelectionBoard` 和 `ModeDisclosure`

### `exploratory_snapshot`

低保证模式。

适用条件：

- 预算很低
- 需要先快速摸清候选空间
- 当前环境不适合完整 deliberation

要求：

- 明确声明未运行的 phase
- 返回结构化 candidate / objection hints
- 不得伪装成 full structural closure

## 3. 最小降级梯子

按这个顺序处理：

1. `retry`
2. `rehydrate in replay mode`
3. `reduce active speakers`
4. `reduce candidate set`
5. `return degraded AttemptResult`

不要一上来就整体降级。

## 4. 最小 runtime 记录

### `DiscussionState`

```yaml
active_phase: frame | blind_diverge | critique | repair | judge | verify
candidate_status:
  active: []
  rejected: []
  selected: []
round_budget_used: 0
termination_reason: ""
```

### 参与者状态

```yaml
participant_id: ""
agent_id: ""
runtime_mode: persistent | replay | degraded
phase_roles:
  - generate | critique | repair | judge | verify
round_last_seen: 0
your_last_position: ""
pending_points: []
status: active | stale | replaced | retired
replacement_of: ""
```

### 候选物状态

```yaml
candidate_id: ""
originator: ""
version: 1
status: active | merged | rejected | selected
parent_candidates: []
latest_thesis: ""
latest_killer_risk: ""
latest_fastest_test: ""
top_objections: []
current_owner: ""
```

## 5. Phase Gates

默认只强制两道 gate：

### before first critique

- 有可区分 candidate set
- 每个 shortlisted candidate 至少具备：
  - `candidate_id`
  - `thesis`
  - `novelty_basis`
  - `killer_risk`
  - `fastest_test`
- 若是 `full_deliberation`，必须已有 `SelectionBoard`

### before final recommendation

- 已有用户可见 artifact
- 顶级 objection 已标记为 `open / addressed / mitigated / closed`
- 已输出 `ModeDisclosure`

注意：

- 不要把中间所有 phase 都做成同等硬阻断
- `blind_diverge` 允许 frontier candidates 不完整，但不允许不可区分
- `exploratory_snapshot` 不走 full gate，但必须显式声明 guarantee 降级

## 6. Minimum Inner Round Packet

主持人给某一位参与者发送新一轮消息时，默认使用：

```yaml
active_phase: ""
round_id: 0
objective_spec: {}
quality_rubric: {}
verified_facts: []
your_last_position: ""
what_changed_since_last_round: ""
named_targets: []
expected_artifact: ""
runtime_mode: persistent | replay | degraded
```

如果当前是 `replay`，再强制补：

```yaml
candidate_lineage_in_scope: []
```

## 7. 遥测规则

至少记录：

- 为什么当前轮选择这几个 active speakers
- 本轮新增了什么 `delta`
- 哪些候选被保留、淘汰或合并
- 为什么结束或换相位

默认阈值：

- 连续两轮 `delta` 很低时，必须换相位、换对象或停止
- `repetition_ratio` 明显升高时，必须减少发言者或收缩候选集
- 用户要“成熟方案”且问题高风险时，结束前优先跑 `verify`

## 8. Effort Budget Enforcement

`effort_budget` 不是装饰字段。

主持人应按预算约束：

- `low`
  - 候选物不要超过 `3`
  - 多轮不要超过 `2`
- `medium`
  - 候选物不要超过 `5`
  - 多轮不要超过 `4`
- `high`
  - 候选物可以到 `7`
  - 多轮最多 `5`

## 9. 默认透明度提示

以下字段默认必须对用户可见：

- `mode`
- `content_origin`
- `degrade_event`
- `synthesis_used`

当发生降级或特殊 pass 时，主持人显式提示：

- `mode: persistent`
- `mode: replay`
- `mode: degraded`
- `pass: consistency_check`
- `pass: verification`
