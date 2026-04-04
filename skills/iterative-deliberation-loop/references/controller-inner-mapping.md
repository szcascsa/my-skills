# Controller-to-Inner Contract Mapping

这个文件定义 outer minimal controller 如何复用 inner `subagent-discussion` contract。

目标：
- 明确哪个字段 authoritative
- 明确哪些对象可以直接复用
- 避免 outer 和 inner 各自维护半套真相

## Mapping Table

| Outer state | Inner source | authoritative 字段 | 说明 |
| --- | --- | --- | --- |
| `CampaignFrame.goal` | `ObjectiveSpec.goal` | outer | outer 决定整个 campaign 目标；inner 只消费当前 attempt 视角 |
| `CampaignFrame.desired_artifact` | `ObjectiveSpec.final_artifact` | outer | inner 不应擅自改 artifact 类型；改了就要走 `RubricChallengeWindow` |
| `CampaignFrame.stop_condition` | `ObjectiveSpec.stop_condition` | outer | outer 定义 campaign stop，inner 定义本轮 attempt 停止点 |
| `CampaignFrame.review_rubric` | `QualityRubric.axes / fail_conditions` | outer preset + inner instance | outer 决定 preset；inner 生成本轮可执行 rubric |
| `AttemptIntent.objective_mode` | `ObjectiveSpec.objective_mode` | outer | outer 选 mode，inner 按 mode 运行 |
| `AttemptIntent.focus_question` | `RoundTask.current_question` | outer | inner 的 round questions 必须服务当前 attempt focus |
| `AttemptIntent.review_focus` | `RoundTask.expected_artifact` + reviewer prompt | outer | reviewer 必须显式收到这一项 |
| `CandidateLineage` | `CandidateArtifact.candidate_id` | outer | inner 产出 candidate，outer 决定跨 attempt lineage |
| `DecisiveObjectionQueue` | `CritiqueMemo.main_objection` | outer | inner 产出 objection；outer 只提升 route-changing family |
| `Board Snapshot` | `SelectionBoard` | inner | 用户可见 shortlist 直接复用 inner board |
| `Mode Disclosure` | `ModeDisclosure` | inner | outer 默认透传，不自行杜撰 |
| `DecisionDelta` | `ModeratorSummary.decision_state` | outer | inner 可总结，但外层最终裁决以 `DecisionDelta` 为准 |

## Adapter Rules

### 1. inner 没返回，不要伪造 rich object

如果 inner 没返回 `SelectionBoard` 或 `ModeDisclosure`：
- outer 可以明确写缺失
- 不要假装这些对象已经存在

### 2. outer 只提升 route-changing 内容

`CritiqueMemo` 里的所有问题，不会自动进入 `DecisiveObjectionQueue`。
只有会改变下一控制决策的 objection 才值得提升。

### 3. lineage 由 outer 统一管理

inner 的 `candidate_id` 是本轮产物。
跨 branch、跨 attempt 的 lineage 归 outer 管。

### 4. packet 分层恢复

- `Campaign Packet`：显式序列化 outer `CampaignFrame / CandidateLineage / DecisiveObjectionQueue / DecisionDelta / BranchStatus / verification_state`
- `Attempt Resume Packet`：恢复 inner `ObjectiveSpec / FactSheet / SelectionBoard / ModeDisclosure`，并携带 `AttemptIntent` 的关键 routing 字段

### 5. named targets 只在 inner 恢复层出现

`named_targets` 属于 inner round routing，不是 outer always-on anchor。
只在 `Attempt Resume Packet` 或 replay 场景里恢复。
