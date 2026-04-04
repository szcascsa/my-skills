# Branch Decisions

主持人只能从有限动作集合里选。

这不是形式主义，而是为了防止多轮讨论滑回“继续聊聊看”。

动作边界的详细字段规则见 [action-state-transitions.md](action-state-transitions.md)。

如果 `verification_state.status = blocked`：
- 可以记录计划中的 `chosen_action`
- 但不要执行任何一个 action，直到验证结果被消费或风险被显式 `accepted_risk`

## Decision Set

### `continue_same_roster`

适合：
- objection 很聚焦
- 当前 roster 仍然是最合适的一组人
- 同一 branch、同一 thesis family 下，再来一轮定向回应大概率能解决

必须满足：
- `structural_progress` 不是连续 `2` 轮 `none`
- 至少有一个明确 target objection
- `VerificationGate` 没有阻塞

不要用于：
- 已经连续两轮低增量
- 缺的是新 lens，而不是补充说明
- 本轮想偷偷改 `thesis / framing / success_condition`

### `revise_same_roster`

适合：
- reviewer 指出的问题需要非平凡修订
- 原 branch 仍值得保留
- 这轮会围绕同一 candidate family、同一 success condition 做更深修补

必须满足：
- artifact 有核心价值，但还没到 branch-worthy 的改道程度
- `AttemptIntent` 明确写了要关闭哪个 objection 或修哪个结构缺口

不要用于：
- 只是把 branch-worthy 的新 thesis 伪装成“修订版”
- `success_condition` 已经实质变化
- 同一 objection family 连续两轮只被 `addressed` 没有真实 closure 迹象

### `retire_and_respawn`

适合：
- 当前 roster 明显 fixation
- 缺少关键 lens
- 同一组人不断在同一 framing 里打转

典型动作：
- 关闭旧 subagents
- 重设计 roster
- 继承 `CampaignFrame`、当前 branch 和未关闭的 decisive objections
- 明确说明新 roster 预计会改变哪个 objection state

不要用于：
- 只是普通 wording 修补
- 根因是缺证据而不是缺视角

### `branch_new_direction`

适合：
- reviewer 发现一个不应只靠修补解决的替代方向
- 不同路线都还有价值，不能过早混合
- `thesis / framing / success_condition` 中至少有一项发生了实质变化

分支时必须记录：
- `parent_attempt_id`
- 新 branch 的核心 thesis
- 为什么不是原 branch 内修补
- 新 branch 准备挑战或关闭哪个 decisive objection

### `stop_and_summarize`

适合：
- 当前推荐已足够成熟
- 主要 objection 已 `closed` 或 `accepted_risk`
- 继续的边际价值低
- 或预算已经打满

不要用于：
- 仍有未 disposition 的高优先级 objection
- 只是因为讨论累了

## Decision Table

| Decision | 保留当前 roster | 关闭 subagents | 开新 branch | 典型用途 |
| --- | --- | --- | --- | --- |
| `continue_same_roster` | 是 | 否 | 否 | 窄焦点快速回应 |
| `revise_same_roster` | 是 | 否 | 否 | 同 branch 深修补 |
| `retire_and_respawn` | 否 | 是 | 否 | 换视角、换阵容 |
| `branch_new_direction` | 视情况 | 视情况 | 是 | 路线分化比较 |
| `stop_and_summarize` | 否 | 是 | 否 | 收束交付 |

## Anti-Loop Rule

出现以下任一情况时，不要再选 `continue_same_roster`：
- 连续两轮 `no structural progress`
- reviewer 已明确指出缺的是新 lens
- objection 本质上否定了当前 framing

出现以下任一情况时，`revise_same_roster` 也要高度警惕：
- 同一 objection family 连续两轮只被 `addressed`
- 本轮想改 `thesis / framing / success_condition`
- reviewer 指出的是 branch drift，不是本 branch 内的局部缺口

## 主持人裁决模板

```text
【主持】Decision
- chosen_action：...
- reason：...
- state_changes：...
- roster_change：keep / add specialist / replace / close
- branch_change：none / create / close
- next_focus：...
```
