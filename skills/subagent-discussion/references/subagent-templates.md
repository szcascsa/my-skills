# Subagent Templates

这份文件只定义 inner discussion engine 自己拥有的对象模板。

`AttemptPacket` / `AttemptResult` 的 authoritative 定义见：

- [attempt-boundary.md](../../iterative-deliberation-loop/references/attempt-boundary.md)
- [lifecycle-ownership.md](../../iterative-deliberation-loop/references/lifecycle-ownership.md)

## 1. `ParticipantCard`

每个参与者先被压缩成一个稳定卡片，再生成 subagent。

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
```

关键规则：

- `planner` 与 `critic` 默认分离
- `anchor_confidence = low` 且用户没有坚持人物锚点时，默认退回 `role`
- `do_not_claim` 用来阻止把不确定内容伪装成真实人物的确定观点

## 2. `ObjectiveSpec`

主持人在 attempt 开场前固定目标物。

```yaml
goal: ""
objective_mode: idea_funnel | architecture_synthesis | pressure_test | decision_debate | sensemaking
discussion_format: roundtable | free_discussion | debate | expert_council | red_team
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

## 3. `FactSheet`

如果议题依赖外部事实、最新信息、论文状态、政策、价格或产品现状，先发一个最小 `FactSheet`：

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

- `status = verified` 表示已做事实核实
- `status = unverified` 时，`sources_or_note` 应显式包含 `no fact verification performed`

## 4. `QualityRubric`

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

按 `objective_mode` 裁剪 axes，不必每次全开。

## 5. `DiscussionState`

```yaml
active_phase: frame | blind_diverge | critique | repair | judge | verify
candidate_status:
  active: []
  rejected: []
  selected: []
round_budget_used: 0
termination_reason: ""
```

默认不要把它扩成 transcript 级日志系统。

## 6. `CandidateArtifact`

```yaml
candidate_id: ""
originator: ""
thesis: ""
why_new: ""
novelty_basis: ""
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
- `novelty_basis` 说明新意来自哪类变化
- `killer_risk` 指出最可能失败的位置
- `fastest_test` 指出最快速的判别实验

## 7. `CritiqueMemo`

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

- 攻击 strongest stated thesis
- 优先指出决定性风险
- 给出 `salvage_path`

## 8. `SelectionBoard`

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

- 明显 outlier 默认保留至少一个
- 淘汰必须说明原因
- 被保留的 outlier 后续必须经历一次完整 `CritiqueMemo`

## 9. `ModeDisclosure`

用户可见的透明度提示。

```yaml
mode: persistent | replay | degraded
content_origin: participant-originated | moderator-synthesized | mixed
degrade_event: ""
synthesis_used: yes | no
```

不要再把 `spokesperson` 或 `degraded_synthesis` 当成并列 mode。

## 10. spawn prompt 模板

```text
You are one participant in a moderated multi-agent discussion.

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
- If you critique, attack the strongest version you can infer.
- If you are anchored to a real or historical figure, emulate worldview and reasoning style by paraphrase only.
- Do not fabricate direct quotes or precise factual claims you are unsure about.
- Respond to the moderator's task for this round, not to every possible point.

Return:
phase: frame | blind_diverge | critique | repair | judge | verify
participant: {display_name}
move: claim | challenge | rebut | refine | bridge | synthesize | question | judge | verify
delta_from_existing: <what is materially new>
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

## 11. `RoundTask` / minimum inner runtime packet

主持人给某一位参与者发送新一轮消息时，建议使用：

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

Named targets:
{named_targets}

Your last position:
{your_last_position}

What changed since last round:
{what_changed_since_last_round}

Points you must address:
{points_you_must_address}

Stop rule:
{stop_rule}

Runtime mode:
{runtime_mode}

Constraints:
- Focus on one or two decisive points.
- Add new structure, not repetition.
- State what would change your mind.
- If generating, produce non-overlapping candidates.
- If critiquing, include severity and salvage path.
```

这个 packet 只负责 inner round routing，不替代 `AttemptPacket`。

## 12. 主持人综合模板

```text
【主持】Mode Disclosure
- mode：persistent / replay / degraded
- content_origin：...
- degrade_event：...
- synthesis_used：...

【主持】Current Board
- 保留候选：...
- 已淘汰候选：...
- 当前 outlier：...

【主持】Decision State
- 当前推荐：...
- 最大 objection：...
- objection disposition：open / addressed / mitigated / closed
- 哪些证据会改变判断：...

【主持】Next Step
- ...
```

## 13. 人数扩展策略

当人数较多时，做分层，不做机械扩容：

- `core roster`
  当前轮可能发言的核心参与者
- `reserve roster`
  后续按相关性唤起

推荐规则：

- `N <= 4`：可近似全员参与
- `5 <= N <= 8`：使用“全员首轮 + 后续活跃子集”
- `N > 8`：使用“分组 + 代表轮换”
