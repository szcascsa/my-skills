# Runtime Modes

这个文件只描述 runtime / orchestration 语义，不重复讨论协议本身。

## 1. 模式分层

### `persistent_agents`

优先模式。

适用条件：
- 当前环境支持创建常驻 subagent
- 主持人可以在多轮中继续向同一 agent 投递

要求：
- 保留 `participant_id -> agent_id` 映射
- 每轮记录最近一次已知立场、未回应问题与当前 phase role
- 维护候选物 lineage，避免后续失去上下文

### `stateless_round_replay`

第一层降级。

适用条件：
- 当前环境不能稳定维持常驻 agent
- 或常驻 agent 已失活，需要重建

要求：
- 每轮重发最小前情包
- 至少带上：
  - `topic`
  - `objective_spec`
  - `quality_rubric`
  - `verified_facts`
  - `candidate_lineage_in_scope`
  - `your_last_position`
  - `what_changed_since_last_round`
  - `named_targets`
  - `points_you_must_address`

### `single-shot_fallback`

最后一层降级。

适用条件：
- 无法维持多轮
- 或当前轮已有足够信息，继续细分收益很低

要求：
- 主持人显式告知当前已降级
- 输出以综合和 artifact 为主，不再伪装成完整多轮讨论

## 2. 最小降级梯子

按这个顺序处理：

1. `retry`
2. `rehydrate in replay mode`
3. `swap to reserve roster or spokesperson`
4. `reduce candidate set`
5. `degraded synthesis`

不要一上来就整体降级。

## 3. 最小 runtime 记录

主持人至少维护以下三类状态。

### 最小强制 `DiscussionState`

这是所有运行模式都必须维护的最小内部状态核心：

```yaml
active_phase: frame | blind_diverge | critique | repair | judge | verify
candidate_status:
  active: []
  rejected: []
  selected: []
round_budget_used: 0
termination_reason: ""
```

不要默认维护全量日志。
只有在 replay / degraded / audit 需要时再扩展。

### 参与者状态

```yaml
participant_id: ""
agent_id: ""
runtime_mode: persistent_agents | stateless_round_replay | single-shot_fallback
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

### 会话追踪状态

```yaml
objective_mode: ""
discussion_format: ""
active_phase: frame | blind_diverge | critique | repair | judge | verify
rounds_used: 0
round_budget: 0
active_speakers: []
active_speaker_reason: ""
repetition_ratio: 0.0
last_round_delta: ""
termination_reason: ""
```

## 4. 默认 `PhaseGate`

默认只强制两道 gate：

1. **before first critique**
   - 有可区分 candidate set
   - 每个候选至少具备 `candidate_id`、`why_new`、`novelty_basis`、`killer_risk`
   - `SelectionBoard` 已初始化

2. **before final recommendation**
   - 已有用户可见 artifact
   - 顶级 objection 已有 disposition
   - 已输出 `ModeDisclosure`

注意：
- 不要把中间所有 phase 都做成同等硬阻断
- `blind_diverge` 允许 frontier candidates 不完整，但不允许不可区分

## 5. Minimum Runtime Packet

launch / replay / replacement 默认都复用同一套最小 packet：

```yaml
active_phase: ""
round_id: 0
objective_spec: {}
quality_rubric: {}
verified_facts: []
candidate_lineage_in_scope: []
your_last_position: ""
what_changed_since_last_round: ""
named_targets: []
expected_artifact: ""
runtime_mode: persistent_agents | stateless_round_replay | single-shot_fallback
```

如果 packet 缺少 `candidate_lineage_in_scope` 或 `named_targets`，优先补齐，不要直接推进。

## 6. 主持人遥测规则

至少记录这些信息：
- 为什么当前轮选择这几个 active speakers
- 本轮新增了什么 `delta`
- 哪些候选被保留、淘汰或合并
- 为什么结束或换相位

默认阈值：
- 若连续两轮 `delta` 很低，必须换相位、换对象或停止
- 若 `repetition_ratio` 明显升高，必须减少发言者或收缩候选集
- 若用户要“成熟方案”，且问题高风险，结束前优先跑 `verify`

## 7. Effort Budget Enforcement

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

超出预算前，先问自己：
- 新一轮会带来新的区分度吗
- 还是只是在重复已有判断

## 8. 默认透明度提示

以下字段默认必须对用户可见：
- `mode`
- `content_origin`
- `degrade_event`
- `synthesis_used`

详细 lineage 只在以下情况强制展开：
- `replay`
- `spokesperson`
- `degraded_synthesis`
- 主持人替代了缺失 participant work

## 9. 用户可见提示

当发生降级或特殊 pass 时，主持人必须显式提示：
- `mode: persistent`
- `mode: replay`
- `mode: spokesperson`
- `mode: degraded_synthesis`
- `pass: consistency_check`
- `pass: verification`
