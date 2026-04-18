# Action-State Transitions

这个文件定义五个 outer actions 到底允许改什么，不允许改什么。

核心原则：

- 同一 branch 内允许修补，不允许偷换 thesis family
- 一旦改 `thesis / framing / success_condition`，先过 `BranchGate`
- `DecisionDelta.state_changes` 必须和动作语义一致

## Verification Blocker

如果 `verification_state.status = blocked`：

- 五个 action 都只能作为计划动作写入 `DecisionDelta`
- 不要实际执行 `continue / revise / respawn / branch / stop`
- 先消费 `verification_result`，再清除 blocker 或改写决策

## Transition Table

| Action | 允许的变化 | 使用前必须满足 | 禁止的变化 | 必须写进 `state_changes` |
| --- | --- | --- | --- | --- |
| `continue_same_roster` | 缩窄问题、追加定向回应、推进同一 objection 的下一次尝试 | 当前 roster 仍合适；存在 target objection；`verification_state.status` 不是 `blocked` | 改 thesis family；改 framing；改 success condition | 目标 objection、下一轮 focus、为什么同一 roster 仍有效 |
| `revise_same_roster` | 同一 branch 内做更深修补、补结构缺口、收窄适用边界 | 当前 artifact 有 salvageable core；`AttemptIntent` 指向明确修补目标；`verification_state.status` 不是 `blocked` | 用“修订”掩盖新 thesis、新 framing 或新 success condition | 被修的 candidate、被处理的 objection、哪些边界被收紧 |
| `retire_and_respawn` | 更换阵容、替换视角、重设 inner runtime request | 当前 roster fixation；缺关键 lens；问题不是单纯缺措辞；`verification_state.status` 不是 `blocked` | 把普通修补包装成 respawn；丢掉未关闭 objections | 关闭哪些 participants、继承哪些 state、新 roster 打算改变什么 |
| `branch_new_direction` | 新 thesis family、新 framing、新 success condition、新 candidate family | 已说明 `why not revise`；新路线可区分且值得单独验证；`verification_state.status` 不是 `blocked` | 只换 wording 就开 branch；开 branch 但不写 parent | 新 branch、parent attempt、核心 thesis、要挑战的 objection |
| `stop_and_summarize` | 关闭 branch、收束建议、保留 `Campaign Packet` | 主要 objection 已 `mitigated / closed / accepted_risk`；或预算到顶且继续价值低；`verification_state.status` 不是 `blocked`，或风险已显式 `accepted_risk` | 留下未 disposition 的高优 objection；临时疲劳式停止 | 最佳方案、关闭理由、残余风险、下一步验证建议 |

## Semantic Drift Check

决定 `continue` 还是 `revise` 前，先问：

1. 这轮是否还在同一 thesis family 上？
2. `success_condition` 是否还是同一个？
3. 这轮是不是只在修补原 objection，而不是换新问题？
4. 如果把这轮摘要给新主持人，对方会不会认为这其实已经是新 branch？

只要第 `1-3` 条有一条为否，或第 `4` 条为是，就优先考虑 `branch_new_direction`。

## Minimal Decision Discipline

- `continue_same_roster` 适合窄焦点快速回应
- `revise_same_roster` 适合同 branch 深修补
- `retire_and_respawn` 适合换视角，不适合补 wording
- `branch_new_direction` 适合真实路线分化
- `stop_and_summarize` 适合收束，不适合逃避高优 objection
