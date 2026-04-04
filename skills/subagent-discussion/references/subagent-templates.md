# Subagent Templates

## 1. `ParticipantCard` 模板

每个参与者都应先被压缩为一个稳定卡片，再生成 subagent。

```yaml
display_name: ""
participant_mode: role | person | hybrid
anchor_person: ""
core_position: ""
lens: ""
phase_roles:
  - generate | critique | repair | judge | verify
selection_rationale: ""
non_overlap_reason: ""
blind_spots: []
challenge_targets: []
collaboration_targets: []
speaking_style: ""
evidence_bar: ""
novelty_bias: low | medium | high
anchor_confidence: low | medium | high
do_not_claim: []
stay_alive_until_discussion_ends: true
```

字段说明：
- `phase_roles`
  - 该参与者主要在哪些相位被激活
- `novelty_bias`
  - 该参与者更偏向保守筛选还是大胆探索
- `selection_rationale`
  - 为什么它必须在场
- `non_overlap_reason`
  - 为什么它不应与其他参与者合并
- `anchor_confidence`
  - 仅用于 `person` / `hybrid`
  - 若为 `low`，且用户未坚持人物锚点，默认退回 `role`
- `do_not_claim`
  - 哪些内容不能被伪装成该人物的确定观点

## 2. `ObjectiveSpec` 模板

主持人在开场前先固定目标物。

```yaml
goal: ""
objective_mode: idea_funnel | architecture_synthesis | pressure_test | decision_debate | sensemaking
discussion_format: roundtable | free_discussion | debate | expert_council | red_team
launch_mode: minimal_launch_lane | explicit_setup
final_artifact: ""
success_criteria:
  - ""
stop_condition: ""
evaluation_axes:
  - ""
fact_dependency: low | medium | high
effort_budget: low | medium | high
allowed_tools_or_sources: []
out_of_scope: []
```

## 3. `FactSheet` 模板

如果议题依赖外部事实、最新信息、论文状态、政策、价格或产品现状，主持人先发一个最小 `FactSheet`：

```yaml
status: verified | unverified
facts:
  - ""
unknowns:
  - ""
source_quality: low | medium | high
unresolved_claims:
  - ""
sources_or_note:
  - ""
```

说明：
- `status=verified` 表示已做事实核实
- `status=unverified` 时，`sources_or_note` 应显式包含 `no fact verification performed`

## 4. `QualityRubric` 模板

把“好方案”定义清楚，不要只让参与者自由发挥。

```yaml
axes:
  novelty: 0-5
  feasibility: 0-5
  evidence_strength: 0-5
  risk_exposure: 0-5
  reversibility: 0-5
fail_conditions:
  - ""
notes:
  - ""
```

轴可以按 `objective_mode` 裁剪，不必每次都全开。

## 5. `DiscussionState` 模板

这是主持人必须维护的最小内部状态核心。

```yaml
active_phase: frame | blind_diverge | critique | repair | judge | verify
candidate_status:
  active: []
  rejected: []
  selected: []
round_budget_used: 0
termination_reason: ""
```

默认不要把它做成完整日志系统。
若发生 replay / degraded 路径，再按 runtime 文档扩展。

## 6. `CandidateArtifact` 模板

这是讨论中的核心对象，不是自由文本意见。

```yaml
candidate_id: ""
originator: ""
thesis: ""
why_new: ""
novelty_basis: ""
readiness: frontier | standard
assumptions:
  - ""
killer_risk: ""
fastest_test: ""
merge_candidates: []
claim_types:
  - verified_fact | inference | speculation
```

要求：
- `why_new` 说明与已有候选的差异
- `novelty_basis` 说明新意具体来自哪一种变化，而不是只说“不同”
- `killer_risk` 说明最可能失败的位置
- `fastest_test` 说明最快速的判别实验或验证方式

## 7. `CritiqueMemo` 模板

结构化批评优先于情绪化反驳。

```yaml
reviewer: ""
target_candidate: ""
main_objection: ""
severity: low | medium | high
evidence_needed: ""
failure_mode: ""
salvage_path: ""
what_would_change_my_mind: ""
```

要求：
- 攻击目标的最强版本
- 优先指出决定性风险，而不是边缘吹毛求疵
- 给出 `salvage_path`，不要只唱反调

## 8. `SelectionBoard` 模板

主持人每轮应维护候选生死状态。

```yaml
survivors:
  - candidate_id: ""
    keep_reason: ""
rejected:
  - candidate_id: ""
    drop_reason: ""
outlier_kept:
  - candidate_id: ""
    keep_reason: novelty | evidence_asymmetry | boundary_case_coverage
next_focus:
  - ""
```

要求：
- 如存在明显 outlier，默认保留至少一个进入下一轮
- 淘汰必须说明原因
- 被保留的 outlier 后续必须经历一次完整 `CritiqueMemo`

## 9. `PhaseGate` 模板

默认只硬化两道 gate。

```yaml
before_first_critique:
  requires:
    - distinguishable_candidate_set
    - candidate_fields_present
    - selection_board_initialized
before_final_recommendation:
  requires:
    - visible_artifact_block
    - objection_disposition
    - mode_disclosure
```

不要把中间所有 phase 都做成强阻断。

## 10. `ModeDisclosure` 模板

用户可见的默认透明度提示。

```yaml
mode: persistent | replay | spokesperson | degraded_synthesis
content_origin: participant-originated | moderator-synthesized | mixed
degrade_event: ""
synthesis_used: yes | no
```

默认轻量显示以上 4 项。
只有在 replay / degraded 路径下，再升级为详细 lineage。

## 11. spawn prompt 模板

在创建每个常驻 subagent 时，可按下列结构发 prompt：

```text
You are one persistent participant in a moderated multi-agent discussion.

Participant card:
{participant_card}

Objective spec:
{objective_spec}

Quality rubric:
{quality_rubric}

Topic:
{topic}

Ground rules:
- Stay faithful to your participant card.
- Add delta, not repetition.
- If you are anchored to a real or historical figure, emulate their worldview,
  priorities, and reasoning style by paraphrase. Do not fabricate direct quotes
  or factual claims you are unsure about.
- You are a persistent participant. After each response, stay available for the
  next round. Do not assume the discussion is finished unless the moderator says so.
- Respond to the moderator's task for this round, not to every possible point.
- Prefer decisive disagreement over generic agreement.
- If you critique, attack the strongest version you can infer.

Return each turn in this structure:
phase: frame | blind_diverge | critique | repair | judge | verify
participant: {display_name}
move: claim | challenge | rebut | refine | bridge | synthesize | question | judge | verify
delta_from_existing: <what is materially new in this turn>
response: <main response>
targets: [who you directly responded to]
output_type: candidate | critique | repair | judgment | verification
claim_types:
- verified_fact | inference | speculation
killer_risk: ...
evidence_that_would_change_my_mind: ...
fastest_test: ...
merge_with_candidates: []
reject_candidates: []
next_question: ...
```

## 12. `RoundTask` / minimum runtime packet 模板

主持人给某一位参与者发送新一轮消息时，建议使用这种结构：

```text
Phase:
{phase}

Round:
{round_id}

Objective:
{objective}

Current question:
{question}

Expected artifact:
{expected_artifact}

Verified facts:
{verified_facts_or_no_fact_verification}

Shared digest:
{digest}

Candidate lineage in scope:
{candidate_lineage_in_scope}

You should respond to:
{named_targets}

Your job in this round:
{response_mode}

Your last position:
{your_last_position}

What changed since last round:
{what_changed_since_last_round}

Points you must address:
{points_you_must_address}

Stop rule:
{stop_rule}

Allowed tools or sources:
{allowed_tools_or_sources}

Out of scope:
{out_of_scope}

Runtime mode:
{runtime_mode}

Constraints:
- Focus on one or two decisive points.
- Directly engage at least one named target when targets exist.
- Add new structure, not repetition.
- State what would change your mind.
- If generating, produce non-overlapping candidates.
- If critiquing, include severity and salvage path.
```

这个 packet 是 launch / replay / replacement 的统一最小合同。
不要在 replay 时退化成只发一个自由文本摘要。

## 13. 主持人综合模板

主持人每轮结束后建议至少输出：

```text
【主持】Session Setup
- 议题：...
- 目标：...
- objective_mode：...
- discussion_format：...
- 当前 phase：...
- 事实状态：...

【主持】Mode Disclosure
- mode：...
- content_origin：...
- degrade_event：...
- synthesis_used：...

【主持】Current Board
- 保留候选：...
- 已淘汰候选：...
- 当前 outlier：...

【主持】本轮推进
- 新增信息：...
- 被加强的候选：...
- 被削弱的候选：...

【主持】Decision State
- 当前推荐：...
- 最大 objection：...
- objection disposition：accepted / mitigated / unresolved
- 哪些证据会改变判断：...

【主持】Next Step
- ...
```

## 14. 人数扩展策略

当人数较多时，不要机械扩容同轮发言人数，而要做分层：

- `core roster`: 核心参与者，当前轮可能发言
- `reserve roster`: 后续按相关性唤起
- `camp spokesperson`: 当同阵营人数过多时，由代表先发言，其他人只在补强或拆解时登场

推荐规则：
- `N <= 4`：可近似全员参与
- `5 <= N <= 8`：使用“全员首轮 + 后续活跃子集”
- `N > 8`：使用“阵营/专题分组 + 代表轮换”
