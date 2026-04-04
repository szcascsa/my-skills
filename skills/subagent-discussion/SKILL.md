---
name: subagent-discussion
description: Use when a user wants multiple subagents or roles to discuss, debate, review, pressure-test, or iteratively refine ideas, designs, or decisions on one topic, including requests like "圆桌讨论", "自由讨论", "辩论", "专家会诊", "多角色评审", "继续这场讨论", "帮我深挖这个方案", or "讨论出新想法".
---

# Subagent Discussion

用一组 subagents 围绕同一议题进行结构化讨论，由主 agent 担任主持人，负责组局、投递、控场、筛选、收敛与交付。

这个 skill 的目标不是“让几个人各说一段”，而是把多角色讨论变成一个可控的候选方案搜索器：
- 先产生彼此有差异的候选观点、方案或假设
- 再让有针对性的参与者去拆、修、比、选
- 最后产出一个成熟的 artifact，而不是只留下热闹的对话

## Core Principle

优先优化结果质量，而不是会话热闹度。

深度讨论的默认路径不是机械轮询，而是：
`frame -> blind diverge -> critique -> repair -> judge -> verify`

其中：
- `discussion_format` 负责“如何路由互动”
- `objective_mode` 负责“最终要产出什么 artifact、按什么 rubric 判断”
- `phase_pipeline` 负责“本轮处在生成、批评、修订、裁决中的哪一相”

不要把三者混成一个概念。

## 何时使用

当用户出现以下意图时触发：
- 明确说要用 subagents / 多 agents / 多个视角讨论一个问题
- 要求“圆桌讨论”“自由讨论”“辩论”“专家会诊”“多角色评审”“红蓝对抗”
- 想在科研、产品、架构、策略上得到真正新颖或更成熟的方案
- 希望多轮推进，而不是一次性收集答案
- 希望让不同身份、不同学派、不同人物锚点围绕同一主题交锋
- 希望继续上一轮，例如“第二轮”“继续这场讨论”“让 A 回应 B”“不要总结，继续推进”

以下情况不要优先使用本 skill：
- 只是查一个事实、命令、API 用法或单点答案
- 低风险、低不确定、明显模式的小修改
- 用户明确只要单个直接答案，不需要多视角探索

## 默认值

除非用户明确指定，否则先按目标推断，而不是把所有任务都塞进同一套讨论模板。

| 项目 | 默认值 |
| --- | --- |
| objective resolver | 新想法/科研发散 -> `idea_funnel`；架构/方案不确定 -> `architecture_synthesis`；已有方案压测 -> `pressure_test`；命题正反 -> `decision_debate`；复杂问题摸清结构 -> `sensemaking` |
| launch policy | 默认 `minimal_launch_lane`：只要求 `topic + inferred objective_mode + final_artifact`，其余字段按 preset lazy derive |
| discussion format | 由目标决定：默认不用 `free_discussion` 作为终局，而把它当成发散波次 |
| phase pipeline | `frame -> blind diverge -> critique -> repair -> judge` |
| round mode | `multi_round` |
| participant count | `4` |
| participant type | `role-first` |
| close behavior | 讨论结束前不主动关闭 subagents |
| fact policy | 议题依赖不稳定事实时，先发布 `FactSheet`；未核实则显式写 `no fact verification performed` |
| output style | 主持人输出结构化 artifact，而不是只给会话摘要 |

## `role-first` 规则

默认先生成角色阵容，而不是自动使用真实人物或历史人物。

只有在以下任一条件满足时才启用 `person` 或 `hybrid`：
- 用户明确要求使用某些真实人物或历史人物
- 主持人能说明该人物锚点对当前目标有不可替代的价值

启用 `person` 或 `hybrid` 时，必须补全对应的安全字段，见 [subagent-templates.md](references/subagent-templates.md)。
如果 `anchor_confidence=low` 且用户没有坚持人物锚点，默认退回 `role-first`。

## 必要输入

最少只需要用户给出议题。

如果以下信息缺失，优先自动补全，只有确实影响结果时才追问：
- 讨论目标
- 讨论形式
- 产出物类型
- 参与者人数
- 参与者来源：角色 / 真实人物 / 历史人物 / 混合
- 单轮还是多轮
- 是否是续轮讨论

## Minimal Launch Lane

默认不要在 round 0 把所有 canonical 对象都显式摊开。

启动讨论时，优先只锁定 3 个输入：
- `topic`
- `objective_mode`
- `final_artifact`

其余内容默认 lazy derive：
- `discussion_format`
- `evaluation_axes`
- `participant_roster`
- `runtime_mode`
- `stop_condition`

只有在以下情况才显式展开全部控制面：
- 高风险或强事实依赖
- 用户明确要求精细控制
- 讨论进入 replay / degraded / spokesperson 路径

对 `idea_funnel + minimal_launch_lane`，默认再加一条 runtime 约束：
- 正常控制流优先走 `blind_diverge -> shortlist -> delta-collapse check -> optional one-shot rescue -> critique_1`
- 第一次 `critique` 前优先依赖 shortlist 可区分性，而不是先要求 `SelectionBoard` 完整初始化
- 不要默认给首波 generator 大规模分配 `SearchFrame`

## 三层控制面

本 skill 的运行有三层，不要混用：

1. `objective_mode`
   定义目标、rubric 和最终 artifact。
   详见 [objective-modes.md](references/objective-modes.md)。

2. `discussion_format`
   定义主持人如何投递和路由。
   详见 [discussion-formats.md](references/discussion-formats.md)。

3. `phase_pipeline`
   定义当前处在哪个阶段：
   - `frame`
   - `blind_diverge`
   - `critique`
   - `repair`
   - `judge`
   - `verify`

如果用户只说“圆桌讨论”，主持人仍然要再补出 `objective_mode` 与 `phase_pipeline`，不能只开一个聊天局。

## Core Contract

本 skill 的 canonical 对象如下：

| 对象 | 作用 | 最小字段 |
| --- | --- | --- |
| `FactSheet` | 定义事实边界 | `status` `facts` `unknowns` `source_quality` `unresolved_claims` `sources_or_note` |
| `ObjectiveSpec` | 定义目标与停止条件 | `goal` `objective_mode` `discussion_format` `final_artifact` `success_criteria` `stop_condition` `evaluation_axes` |
| `QualityRubric` | 定义判断标准 | `axes` `fail_conditions` |
| `DiscussionState` | 定义内部状态机 | `active_phase` `candidate_status` `round_budget_used` `termination_reason` |
| `ParticipantCard` | 定义参与者边界 | `display_name` `participant_mode` `core_position` `lens` `phase_roles` `evidence_bar` |
| `RoundTask` | 定义某一轮投递 | `phase` `round_id` `objective` `current_question` `named_targets` `expected_artifact` `stop_rule` |
| `CandidateArtifact` | 定义被讨论的候选物 | `candidate_id` `thesis` `why_new` `novelty_basis` `assumptions` `killer_risk` `fastest_test` |
| `CritiqueMemo` | 定义结构化批评 | `reviewer` `target_candidate` `main_objection` `severity` `evidence_needed` `salvage_path` |
| `SelectionBoard` | 定义候选物的保留与淘汰 | `survivors` `rejected` `outlier_kept` `next_focus` |
| `ModeDisclosure` | 定义用户可见透明度提示 | `mode` `content_origin` `degrade_event` `synthesis_used` |
| `ModeratorSummary` | 定义主持人输出 | `session_setup` `phase_summary` `decision_state` `next_step` |

内部 canonical 动作枚举：
- `claim`
- `challenge`
- `rebut`
- `refine`
- `bridge`
- `synthesize`
- `question`
- `judge`
- `verify`

面向用户展示时可以映射为中文动作标签，例如 `陈述`、`质疑`、`反驳`、`修正`、`桥接`、`综合`、`裁决`、`核验`。

## Phase 1: 建立讨论契约

主持人在开始前先定 7 件事：

1. **Topic**
   议题要具体，避免“AI”“教育”“经济”这类无边界大词。

2. **Goal**
   是要找新想法、比较方案、做压力测试、还是摸清结构。

3. **Final artifact**
   先决定最后交付什么：
   - `idea portfolio`
   - `option matrix`
   - `decision memo`
   - `risk register`
   - `perspective map`

4. **Fact status**
   如果议题依赖最新事实、政策、价格、论文、产品状态或外部上下文：
   - 先发布 `FactSheet`
   - 如果尚未核实，则明确写 `no fact verification performed`

5. **Objective mode**
   先按目标定 `objective_mode`，再定 discussion format。
   详见 [objective-modes.md](references/objective-modes.md)。

6. **Discussion format**
   只决定互动方式，不决定最终 artifact。
   详见 [discussion-formats.md](references/discussion-formats.md)。

7. **Stop condition**
   至少给出一条明确终止规则，例如：
   - 产出 `top-3` 候选并完成排序
   - 已有推荐方案，且主要 objection 已被回答
   - 连续两轮新增信息过低

## Phase 2: 设计参与者阵容

每个参与者都要有明确的 `ParticipantCard`，模板见 [subagent-templates.md](references/subagent-templates.md)。

阵容设计先看“阶段职责”，再看“发言风格”。

至少覆盖以下角色函数中的三类：
- `generator` / `planner`
- `critic` / `red`
- `reviser` / `bridge`
- `judge` / `synthesizer`
- `verifier`

关键规则：

1. **先设计张力网络，再设计人数**
   阵容必须形成有效分歧，不是平行观点堆砌。

2. **`planner` 与 `critic` 默认分离**
   不要让同一个参与者一边提案一边给自己背书。

3. **真实/历史人物只模拟思想框架，不伪造引文**
   可以化用其问题意识、推理路径和风格，但不要编造精确引号、年份或著作细节。

4. **人数是变量，但每轮活跃发言者必须控住**
   - `N <= 4`：可近似全员参与
   - `5 <= N <= 8`：首轮可全员亮相，后续每轮只激活 `2-4` 人
   - `N > 8`：使用 `core roster + reserve roster + spokesperson`

5. **不要让所有人每轮都说**
   深度来自定向交锋，不来自机械轮询。

## Phase 3: 启动常驻 subagents

优先使用能维持上下文的 persistent subagents；环境不支持时，降级为 replay 模式。
runtime 语义与降级细节见 [runtime-modes.md](references/runtime-modes.md)。

主原则：
- 能 `persistent` 就 `persistent`
- 不能 `persistent` 就显式 replay 前情
- 局部失败先 `retry`，再 `rehydrate`，再 `swap spokesperson`
- 讨论结束、用户切题或需要重组阵容时才关闭

主持人从这一步开始必须维护一个隐藏的 `DiscussionState`。
它不必完整暴露给用户，但必须在每轮结束后更新，至少包含：
- `active_phase`
- `candidate_status`
- `round_budget_used`
- `termination_reason`

## Phase 4: 主持人循环

主持人的职责是“路由 + 选择 + 收敛”，不是替代所有人发言。

### 默认循环

1. **Frame**
   建立 `FactSheet`、`ObjectiveSpec`、rubric 和 stopping rule。

2. **Blind diverge**
   让 `2-4` 位差异足够大的参与者独立生成候选，不要一开始就互相看答案。
   默认要求：
   - 每人 `1-2` 个候选
   - 常规 lane 下，候选应说明 `why_new`
   - 必须说明 `killer_risk`
   - 必须说明 `fastest_test`

   对 `idea_funnel + minimal_launch_lane`：
   - 首波默认先跑普通 `blind_diverge`
   - 首波候选至少要给出 `novelty_basis`、`killer_risk`、`fastest_test`
   - `why_new` 可以在 shortlist 形成后补写，不要求首波全量写满

3. **Cluster and select**
   主持人去重、聚类、合并近似项，只保留 `2-4` 个值得继续压测的候选。
   如有明显 outlier，默认保留至少 `1` 个，不要过早清洗掉。

   对 `idea_funnel + minimal_launch_lane`，进入第一次 `critique` 前再多做一步 `delta-collapse check`：
   - 只对 shortlist 候选临时补 `nearest_baselines`、`decisive_delta`、`discriminating_test`、`collapse_condition`
   - 若 shortlist 明显塌到同一 baseline 家族，允许一次且仅一次 `SearchFrame rescue wave`
   - rescue 只激活 `1-2` 位 generator，每人只分配 `1` 个 operator，并带 `do_not_reuse`
   - rescue 完成后立刻回到 shortlist，不要把它扩成新 phase

4. **Critique**
   把候选投给最相关的 `critic` / `red team`。
   批评时必须攻击“最强版本”，不得打 strawman。

5. **Repair**
   把结构化 objection 投回 `proposer` / `reviser`，要求修正、缩小适用边界，或明确承认该候选应淘汰。

6. **Judge**
   主持人或指定 `judge` 依据 rubric 排序、筛选、合并，形成推荐或 shortlist。

7. **Verify**
   若问题高风险或强依赖外部事实，在结束前单独做一轮事实 / 假设核验，不要把“谁说得更像对”当作验证。

### 两道硬 Gate

默认不要把每个 phase 都做成硬阻断，但以下两道 gate 必须满足：

1. **进入第一次 `critique` 前**
   必须已有可区分的 candidate set：
   - 至少 `2` 个可区分候选，或显式标注为 `moderator-generated fallback`
   - 常规 lane 下，每个候选至少包含 `candidate_id`、`why_new`、`novelty_basis`、`killer_risk`
   - 常规 lane 下，`SelectionBoard` 已记录当前保留项与 outlier 保留原因

   对 `idea_funnel + minimal_launch_lane` 例外：
   - gate 改成 candidate-driven，而不是 `SelectionBoard`-driven
   - shortlist 中至少有 `2` 个可区分候选；若执行过一次 rescue 仍不足，则显式标记 `rescue_failed`
   - 每个 shortlisted candidate 至少包含 `candidate_id`、`thesis`、`novelty_basis`、`killer_risk`、`fastest_test`
   - 若执行过 `delta-collapse check`，再补 `nearest_baselines`、`decisive_delta`、`discriminating_test`、`collapse_condition`
   - 若某候选需要 `2+` 个 baselines 才能描述，且其区分测试只在完整组合形态下成立，不得仅因与单一 baseline 局部重叠就提前 merge/collapse

2. **输出最终 recommendation / shortlist 前**
   必须已有可见 artifact 与 objection disposition：
   - 输出了对应的 artifact block
   - 顶级 objection 已标记为 `accepted`、`mitigated` 或 `unresolved`
   - 已输出 `ModeDisclosure`

### Phase-Sensitive Acceptance

不要把所有阶段都按同一套完整度检查。

- `blind_diverge`
  只要求候选可识别、可区分、可继续压测，不要求已经成熟
- `critique`
  必须攻击 strongest stated thesis，且包含 `what would change my mind`
- `repair`
  必须明确引用至少一个 prior objection，并说明改了什么
- `judge`
  必须说明 `why not the others` 与仍未关闭的 evidence gap
- `verify`
  必须显式区分 `verified fact`、`inference`、`speculation`

### 关键规则

- 每轮只追一个最值得推进的问题
- 每轮只激活最相关的 `2-4` 位参与者
- 每次投递必须写清 `expected_artifact`
- 每次回应都必须增加 `delta`，而不是复述前文
- 前两轮默认不接受泛泛同意
- 连续两轮低增量时，主持人必须换相位、换对象，或结束

## Effort Budget

讨论深度不是靠无限扩张人数和轮数获得的。

默认 budget：

| 复杂度 | subagents | 候选数 | 轮数 |
| --- | --- | --- | --- |
| `low` | `3-4` | `1-3` | `2` |
| `medium` | `4-6` | `3-5` | `3-4` |
| `high` | `5-8` | `4-7` | `4-5` |

没有明确收益前，不要继续加 agent 或拉长讨论。

## Anti-Collapse Rules

默认开启以下规则，防止讨论塌成低质量共识：

- 前两轮内容性回应中，禁止“我基本同意，只补充一点”这类无新增表达
- 每个候选至少要经历一次真正的独立批评
- 至少保留一个高新颖、低置信的 outlier 到裁决阶段
- 被保留的 outlier 必须至少经历一轮完整 `CritiqueMemo`，不能只做象征性保留
- 事实相关内容必须标注 `verified_fact`、`inference` 或 `speculation`
- `critic` 必须指出“什么证据会让我改判”
- `judge` 必须说明为什么淘汰，而不是只说“综合来看不优”
- 默认要求每个 surviving candidate 写明 `novelty_basis`

## 单轮与多轮

### 单轮

适合快速对照或低预算探索。

做法：
- 压缩为 `frame -> generate -> summarize`
- 输出“候选物矩阵 + 主要分歧 + 建议下一步”

### 多轮

这是默认模式。

做法：
- 先独立生成候选
- 再定向压测
- 再修订和裁决
- subagents 保持常驻，带着前轮记忆进入下一轮

即使用户说“继续自由讨论”，主持人也要保留当前 phase state，不要退回无边界群聊。

## 面向用户的输出协议

默认优先输出 artifact，再补会话摘要。

可见输出建议保持以下结构：

```text
【主持】Session Setup
- 议题：...
- 目标：...
- objective_mode：...
- discussion_format：...
- 事实状态：...

【主持】Mode Disclosure
- mode：persistent / replay / spokesperson / degraded_synthesis
- content_origin：participant-originated / moderator-synthesized / mixed
- degrade_event：...
- synthesis_used：yes / no

【主持】Current Board
- 保留候选：...
- 已淘汰候选：...

【主持】Decision State
- 当前推荐：...
- 推荐原因：...
- 关键风险：...
- 哪些证据会改变判断：...

【主持】Next Step
- ...
```

更具体的 artifact 模板见 [artifact-templates.md](references/artifact-templates.md)。

## 主持人行为约束

- 优先暴露分歧结构，而不是追求表面和谐
- 把注意力放在“谁应当回应谁、哪一个候选值得继续投资源”
- 不要把参与者变成主持人的提词器
- 不要把 `free_discussion` 误当成科研 ideation 的默认终局
- 不要用“谁说得更有气势”替代事实核验或方案验证
- 不要把 summary 写得很漂亮，却不交付可比较的 artifact
- 不要在 gate 未满足时偷偷推进 phase，再用漂亮 artifact 掩盖过程缺失
- 不要把主持人补写的内容伪装成真实 participant turn

## 结束条件

以下任一条件满足即可收束：
- 用户明确要求停止
- 已形成可交付 artifact，且主要 objection 已被回答
- 候选方案已完成排序或筛选
- 连续两轮新增信息过低，继续只会重复

结束时至少输出：
- 最终 artifact
- 当前推荐或 shortlist
- 仍未解决的开放问题
- 如有需要，给出下一步验证建议

## 实施提示

根据需要读取：
- [objective-modes.md](references/objective-modes.md)
- [discussion-formats.md](references/discussion-formats.md)
- [subagent-templates.md](references/subagent-templates.md)
- [artifact-templates.md](references/artifact-templates.md)
- [runtime-modes.md](references/runtime-modes.md)
