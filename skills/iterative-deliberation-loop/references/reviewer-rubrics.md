# Reviewer Rubrics

reviewer 不是第二批 proposer。

reviewer 的职责是用较稳定的 rubric 判断：
- 这次 attempt 是否真的推进了问题
- 主要 objection 是什么
- 下一步应该修补、分支、换阵容还是停止

## State-Aware Input Contract

正式 review 前，至少准备：
- `AttemptIntent`
- `baseline_reference`
- `branch snapshot`
- `decisive objection snapshot`
- 最近一次 `DecisionDelta`
- 当前 rubric

没有这些输入时，最多做轻量点评，不要写正式 `ReviewVerdict`。

## Semi-Frozen Rule

rubric 默认 `semi_frozen`：
- 同一 campaign 内，axes 尽量保持稳定
- 只有问题定义或最终 artifact 发生变化时才改 rubric
- 改 rubric 时必须说明 `why rubric changed`

## 通用核心轴

除非有明确理由，review 至少看这几轴：
- `problem_fit`
- `novelty_or_delta`
- `usefulness`
- `tractability`
- `risk_exposure`
- `evidence_gap`

## Structural Progress Rule

reviewer 必须显式判断 `structural_progress`：

- `none`
  同一 branch 下没有关闭高价值 objection，也没有改变 branch 排序或 stop / branch / respawn 判断
- `local`
  同一 branch 下关闭、缩小或澄清了至少一个 decisive objection，但没有改 thesis family
- `branch_shaping`
  这轮改变了 branch 排序、证明了需要开 branch / respawn / stop，或显式暴露了当前 branch 的不可修补缺陷

不要把“说得更顺”当成 `local`。
不要把“偷偷换 thesis”当成 `branch_shaping` 的正向加分；先看它是否违反了 `BranchGate`。

## Preset: `research_innovation`

适合科研创新、idea discovery、论文方向探索。

优先轴：
- `novelty`
- `insight_density`
- `testability`
- `upside_if_true`
- `killer_risk`
- `evidence_gap`

默认 fail conditions：
- 只是旧想法换说法
- 没有清楚的 `fastest_test`
- 高新颖但没有可检验路径

## Preset: `architecture_design`

适合架构设计和工程方案比较。

优先轴：
- `fit_to_problem`
- `complexity`
- `failure_modes`
- `migration_cost`
- `operational_fit`
- `reversibility`

默认 fail conditions：
- 方案与约束不匹配
- 复杂度上涨但收益不清
- failure mode 没被正面处理

## Preset: `generic_complex_problem`

适合暂时不想过早分类的问题。

优先轴：
- `clarity_of_thesis`
- `decision_relevance`
- `delta_from_previous_attempt`
- `main_risk`
- `open_question_quality`

## Review Verdict Template

```text
【Review Cell】Verdict
- judgment：promising / salvageable / branch-worthy / weak / ready_to_stop
- structural_progress：none / local / branch_shaping
- strongest_value：...
- decisive_objections：...
- objection_disposition：closed / still_open / escalated / accepted_risk
- branch_opportunity：...
- verification_needed：none / helpful / required_before_next_control_decision
- what_would_change_my_mind：...
- suggested_action：continue_same_roster / revise_same_roster / retire_and_respawn / branch_new_direction / stop_and_summarize
```

## Reviewer 行为约束

- 攻击 artifact，不攻击角色设定
- 以 `AttemptIntent` 为主基线，而不是只看 thesis 有没有变
- 指出“什么证据会让我改判”
- 不要只写抽象批评，尽量指出最关键的结构性缺陷
- 不要在 review 阶段偷偷重新发明整套方案，除非明确建议 `branch_new_direction`
- 如果 evidence gap 会改变下一控制决策，明确要求先过 `VerificationGate`，并把 `verification_needed` 视为会写入 controller `verification_state` 的硬信号
