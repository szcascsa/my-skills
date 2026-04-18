# Schema Collapse

这个文件定义旧协议如何收敛到新的 boundary-first contract。

目标只有一个：**不要双轨运行。**

如果旧对象和新对象同时常驻，复杂度只会换个名字回来。

## Old -> New Mapping

| 旧对象 | 新归宿 | 迁移规则 |
| --- | --- | --- |
| `OrchestrationIntent` + `ProblemFrame` | `CampaignFrame` | 把 `goal`、`desired_artifact`、`constraints`、`effort_budget`、`risk_tolerance`、`out_of_scope`、`stop_condition`、`review_rubric` 折叠进一个稳定 frame |
| `ReviewRubric` | `CampaignFrame.review_rubric` | 不再单独常驻成一个 outer object；需要变更时在 `CampaignFrame` 上写 `why rubric changed` |
| `AttemptPlan` | `AttemptIntent` | 保留 `attempt_id`、`branch_label`、`objective_mode`、`focus_question`，补 `intended_delta`、`success_condition`、`review_focus` |
| raw outer-to-inner field injection | `AttemptPacket` | 任何跨层输入统一进入 `AttemptPacket` |
| `AttemptArtifact` | `AttemptResult` | 不再复写成主持人版 outer artifact |
| `DecisionRecord` + `SkillTransitionPlan` | `DecisionDelta` | 决策、状态变化和下一步 focus 合并到一个 object |
| `GlobalCandidateRegistry` | `CandidateLineage` | 只保留 lineage，不保留 full scorecard |
| `FactVerificationLedger` | `verification_state.pending_claims` | 降为 pressure-activated controller state，只记录会改变下一控制决策的 claim |
| `AttemptLedger` + `Attempt Resume Packet` | 更薄的 `AttemptLedger` + `Campaign Packet` | 删除独立 resume 协议；恢复时重新构造 `AttemptPacket` |

## Collapse Rules

### 1. 先归一，再继续

如果恢复旧 packet 或旧会话：

- 先把旧字段归一到 `CampaignFrame / AttemptIntent / CandidateLineage / DecisiveObjectionQueue / DecisionDelta / BranchStatus / AttemptPacket / AttemptResult`
- 再继续下一轮

不要一边保留旧名，一边新增新名。

### 2. 外层不重复 inner 对象

如果 inner engine 已返回结构化对象：

- 外层只记录用于跨 attempt 的 state delta
- 不要再把同一内容重抄成另一份 outer artifact

### 3. verification 不是默认记账层

没有明确决策价值的事实核验：

- 不要进 `verification_state.pending_claims`
- 更不要重建一个常驻 `FactVerificationLedger`

### 4. packet 只在 handoff 边界出现

不要每个 attempt 都生成完整 `Campaign Packet`。
只有跨 session、handoff、respawn 或显式暂停时才强制输出。

## Field Normalization

只保留一套 canonical field names：

- 用 `chosen_action`，不用 `decision`
- `AttemptIntent` 必带 `review_focus`
- `ReviewVerdict` 必带 `strongest_value`
- `DecisionDelta` 统一承载 `state_changes`
- objection status 统一用 `open / addressed / mitigated / closed / accepted_risk / reopened / escalated`
- user-facing mode 统一用 `persistent / replay / degraded`

如果恢复旧 schema，先归一这些字段名，再继续 campaign。
