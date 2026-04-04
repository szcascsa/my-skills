# Review Loop

把一个复杂问题组织成：

`frame campaign -> plan attempt -> run discussion -> review artifact -> decide next action -> update ledger -> repeat or stop`

## Attempt 生命周期

### 1. `frame_campaign`

锁定 `CampaignFrame`：
- `campaign_id`
- `goal`
- `desired_artifact`
- `constraints`
- `stop_condition`
- `review_rubric`
- `complexity`

### 2. `plan_attempt`

每次 attempt 只回答一个更窄的问题。

最小 `AttemptIntent`：
- `attempt_id`
- `parent_attempt_id`
- `branch_label`
- `focus_question`
- `objective_mode`
- `intended_delta`
- `review_focus`
- `success_condition`
- `roster_strategy`

### 3. `run_discussion`

调用 [subagent-discussion](../../subagent-discussion/SKILL.md) 作为 inner engine。

优先只把以下最小信息投进去：
- 当前 `topic`
- 当前 `focus_question`
- 本次 `objective_mode`
- 需要继承的 `carry_forward`
- 需要重点回应的 `open_decisive_objections`
- 本次 `review_focus`
- 本次 `final_artifact`

### 4. `review_artifact`

reviewer cell 审读 attempt artifact，而不是接管整个求解过程。

默认 reviewer cell 人数：
- `low`: `1`
- `medium`: `2`
- `high`: `2-3`

正式 review 前，至少交给 reviewer：
- `AttemptIntent`
- `baseline_reference`
- `branch snapshot`
- `decisive objection snapshot`
- 最近一次 `DecisionDelta`
- 当前 rubric

### 5. `decide_next_action`

只能选有限动作：
- `continue_same_roster`
- `revise_same_roster`
- `retire_and_respawn`
- `branch_new_direction`
- `stop_and_summarize`

动作语义边界见 [action-state-transitions.md](action-state-transitions.md)。

### 6. `update_ledger`

记录：
- 本次 attempt 真正新增了什么
- 哪些 objection 被 introduce / promote / merge / close / reopen / accepted_risk
- 哪些 branch 还活着
- 下一轮要改什么

不要把 transcript 当 ledger。

## Pressure-Activated Modules

### `ExpandedReviewCell`

在以下情况启用更重 reviewer 配置：
- 高风险问题
- reviewer 间分歧大
- 连续两轮低增量
- 当前 attempt 可能触发 `branch / respawn / stop`

### `VerificationGate`

在以下情况优先补证据，而不是继续聊：
- evidence gap 会改变下一控制决策
- 单分支高风险场景里，证据会影响 `revise` vs `respawn`
- 当前最大分歧不是想法，而是事实、约束或 external dependency

### `FullCampaignPacket`

在以下情况输出完整 packet：
- 跨 session
- handoff 给新主持人或新 roster
- 即将 `retire_and_respawn`
- 用户明确暂停

## Preset Sequences

### 1. Research Innovation

适合：
- 科研创新 idea
- 新方法假设
- 论文方向探索

推荐 attempt sequence：
1. `idea_funnel`
   先生成 `3-5` 个非重叠候选
2. `pressure_test`
   压测 shortlist，淘掉脆弱想法
3. `idea_funnel` 或 `pressure_test`
   取决于 reviewer 认为应继续扩散还是修补
4. `stop_and_summarize`
   产出 `top ideas + why now + killer risks + fastest tests`

### 2. Architecture Design

适合：
- 系统架构
- 技术方案取舍
- 复杂工程设计

推荐 attempt sequence：
1. `architecture_synthesis`
   独立提出多个方案
2. `pressure_test`
   专门攻击 failure modes、迁移成本和运维风险
3. `architecture_synthesis`
   对 surviving option 修订或 merge
4. `stop_and_summarize`
   产出 `option matrix + recommendation`

### 3. Ambiguous Problem

适合：
- 问题还没看清结构
- 不确定应该先 ideate 还是先设计

推荐 attempt sequence：
1. `sensemaking`
   先画清冲突、前提与未知
2. `idea_funnel` 或 `architecture_synthesis`
   根据输出切到生成候选或生成设计
3. `pressure_test`
   压测最强候选

## 何时切换 Inner Mode

从 `sensemaking` 切到 `idea_funnel`：
- 已经知道关键冲突，但还没有候选解

从 `idea_funnel` 切到 `pressure_test`：
- shortlist 已经形成，需要挑出脆弱点

从 `architecture_synthesis` 切到 `pressure_test`：
- 已经有推荐，但 reviewer 还不信

从 `pressure_test` 回到 `architecture_synthesis` 或 `idea_funnel`：
- reviewer 指出的是结构性问题，不是局部修补问题

## Budget Guidance

| 复杂度 | outer attempts | inner rounds per attempt |
| --- | --- | --- |
| `low` | `2` | `2-3` |
| `medium` | `3` | `3-4` |
| `high` | `4-5` | `4-5` |

外层的 attempt 预算比内层 rounds 更重要。
如果外层没有明确新增价值，不要只靠加内层 rounds 硬拖。
