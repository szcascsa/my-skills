# Objective Modes

`objective_mode` 决定三件事：
- 要追求什么结果
- 用什么 rubric 判断
- 最后交付什么 artifact

不要把它和 `discussion_format` 混淆。

## 1. `idea_funnel`

**适合**
- 科研创新想法
- 新产品方向
- 开放问题的有用新解法

**默认目标**
- 先发散出非重叠候选
- 再把真正值得继续的想法筛出来

**默认流程**
`frame -> blind_diverge -> critique -> repair -> judge`

其中：
- `blind_diverge` 对应独立生成候选
- 主持人在进入 `critique` 前完成去重与聚类

**推荐角色**
- `generator` x `2-3`
- `critic` x `2`
- `synthesizer` x `1`

**优先 rubric**
- `novelty`
- `usefulness`
- `tractability`
- `evidence_gap`

**默认 artifact**
- `idea portfolio`

**主持人重点**
- 第一轮尽量独立生成，不要互相看答案
- 去重时不要把高新颖 outlier 全清掉
- 每个候选都要带 `killer_risk`、`fastest_test` 和 `novelty_basis`
- 被保留的 outlier 必须至少经历一次完整 `CritiqueMemo`

## 2. `architecture_synthesis`

**适合**
- 系统架构选择
- 技术方案取舍
- 工程设计不确定

**默认目标**
- 比较多个候选设计
- 暴露 failure modes
- 收敛到一个可解释的推荐方案

**默认流程**
`frame -> blind_diverge -> critique -> repair -> judge -> verify`

其中：
- `blind_diverge` 对应 independent proposals
- `repair` 可包含 merge

**推荐角色**
- `planner`
- `reliability critic`
- `operator`
- `cost/perf critic`
- `synthesizer`

**优先 rubric**
- `fit_to_problem`
- `complexity`
- `failure_modes`
- `migration_cost`
- `reversibility`

**默认 artifact**
- `option matrix + recommendation`

**主持人重点**
- 先做独立提案，不要让第一位发言者锚定全场
- 批评阶段优先攻击边界条件和落地成本
- 推荐时必须写 `why not the others`

## 3. `pressure_test`

**适合**
- 已有方案、论文论证、创业点子、上线计划需要压测

**默认目标**
- 让最危险的失败路径提前暴露
- 把脆弱版修成更强版，或明确判死

**默认流程**
`frame -> critique -> repair -> critique -> judge`

其中：
- `frame` 内先把主张压缩成可攻击对象
- 第二个 `critique` 用于修正版复压测

**推荐角色**
- `proposer`
- `red`
- `blue`
- `operator`
- `judge`

**优先 rubric**
- `risk_exposure`
- `severity`
- `salvageability`
- `operational_fit`

**默认 artifact**
- `risk register + patched version`

**主持人重点**
- 要求 `red` 攻击最强版本
- 要求 `blue` 修正，不要辩解
- 要求 `judge` 决定“修补后是否足够强”

## 4. `decision_debate`

**适合**
- 命题正反
- 路线 A vs 路线 B
- 政策或策略取舍

**默认目标**
- 找出哪一边当前证据更强
- 或找出真正决定胜负的关键未决点

**默认流程**
`frame -> blind_diverge -> critique -> repair -> judge`

其中：
- `blind_diverge` 对应 opening claims
- `critique` 对应 cross-examination
- `repair` 对应 rebuttal

**推荐角色**
- `pro`
- `con`
- `cross-examiner`
- `judge`

**优先 rubric**
- `argument_strength`
- `assumption_exposure`
- `evidence_quality`
- `decision_relevance`

**默认 artifact**
- `decision memo`

**主持人重点**
- 先把命题写成可反驳句子
- 每轮强制回应对方最强点
- 裁决时说明“哪些证据会改变当前判定”

## 5. `sensemaking`

**适合**
- 复杂议题摸清结构
- 价值冲突问题
- 尚不适合直接下结论的复杂问题

**默认目标**
- 识别视角、冲突、前提与未知项
- 为后续决策或 ideation 打基础

**默认流程**
`frame -> blind_diverge -> critique -> judge`

其中：
- `blind_diverge` 对应 perspective map
- `critique` 对应 hidden assumption exposure
- `judge` 对应 synthesis

**推荐角色**
- `mapper`
- `skeptic`
- `bridge`
- `synthesizer`

**优先 rubric**
- `coverage`
- `clarity_of_conflict`
- `hidden_assumption_exposure`
- `decision_readiness`

**默认 artifact**
- `perspective map`

**主持人重点**
- 不要强行收敛成结论
- 重点是把“问题长什么样”讲清楚

## 选择建议

| 用户目标 | 推荐 objective_mode |
| --- | --- |
| 想讨论出真正新颖的科研点子 | `idea_funnel` |
| 想收敛出成熟设计方案 | `architecture_synthesis` |
| 已有方案但不放心，想找隐患 | `pressure_test` |
| 想判断正反哪边更强 | `decision_debate` |
| 先把问题结构摸清楚 | `sensemaking` |

## Mode 切换规则

当主持人发现当前 mode 不再合适时，可以切换，但要显式说明。

常见切换：
- `sensemaking -> idea_funnel`
  当问题结构已足够清晰，开始生成候选
- `idea_funnel -> pressure_test`
  当 shortlist 已形成，开始压测
- `architecture_synthesis -> pressure_test`
  当已有首选方案，但风险仍不清楚
- `decision_debate -> verify`
  当表面上已分出强弱，但事实依赖较重
