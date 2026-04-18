# Lifecycle Ownership

先定义权责边界，再定义 packet 和状态机。

如果 `campaign / attempt / discussion / fallback` 四层生命周期没有分清，任何 adapter、status enum 或 resume packet 都会重新漂移。

## Core Rule

只允许一层拥有某一类状态的最终解释权。

外层 controller、内层 discussion engine 和 boundary contract 的分工如下：

- outer controller
  负责 `campaign` 级目标、branch、review gate、ledger 和下一步动作选择
- inner engine
  负责单次 `attempt` 内的讨论执行、candidate 演化、critique / repair / judge / verify
- boundary contract
  负责 outer 和 inner 之间唯一合法的数据交换面

不要让两层同时“半拥有”同一份状态。

## Lifecycle Units

| 生命周期单位 | owner | 进入条件 | 退出条件 | 允许修改的状态 | 序列化责任 |
| --- | --- | --- | --- | --- | --- |
| `campaign` | outer | 用户发起一个需要多 attempt 推进的问题 | `stop_and_summarize` / abandon / explicit pause | `CampaignFrame` `BranchStatus` `CandidateLineage` `DecisiveObjectionQueue` `verification_state` | outer |
| `attempt` | boundary launch by outer, execution by inner | outer 决定发起一次可审讨论 | inner 返回一个 `AttemptResult` | `AttemptIntent` 的执行态、board、objection updates、runtime recommendation | boundary |
| `discussion` | inner | `AttemptPacket.attempt_kind = full_deliberation` 后进入 phase pipeline | full completion / degraded completion / blocked | `ObjectiveSpec` `FactSheet` `CandidateArtifact` `CritiqueMemo` `SelectionBoard` `ModeDisclosure` | inner, then packed into `AttemptResult` |
| `exploratory_snapshot` | inner | `AttemptPacket.attempt_kind = exploratory_snapshot` | snapshot result returned | 候选草案、局部 board、open objection hints | inner, but must be marked lower-guarantee in `AttemptResult` |

## Ownership Table

| 状态家族 | authoritative owner | 说明 |
| --- | --- | --- |
| `CampaignFrame` | outer | 定义整个 campaign 的稳定边界 |
| `AttemptIntent` | outer | 定义本次 attempt 想验证什么，不直接等于 inner phase state |
| `ObjectiveSpec` | inner | attempt 内真正执行的目标和 artifact 形态 |
| `FactSheet` | inner | attempt 内被消费的事实边界 |
| `CandidateArtifact` | inner | 单次 attempt 中被生成、批评、修订的候选物 |
| `CritiqueMemo` | inner | attempt 内结构化批评 |
| `SelectionBoard` | inner | attempt 结束时的 survivor / rejected / outlier 状态 |
| `DecisiveObjectionQueue` | outer | 只保存会改变下一控制决策的 objection family |
| `CandidateLineage` | outer | 只保存跨 attempt / branch 的 candidate 谱系 |
| `DecisionDelta` | outer | controller 对下一步动作的最终裁决 |
| participant liveness inside one attempt | inner | attempt 内是否保活、是否需要 replay |
| roster reuse across attempts | outer | 下一次 attempt 是否复用、respawn 或 stop |

## Transfer Rules

### 1. `campaign -> attempt`

outer 不直接把 controller 私有字段塞进 inner phase 对象。

outer 先构造 `AttemptPacket`，再交给 inner。

### 2. `attempt -> campaign`

inner 不直接改 `BranchStatus`、`CandidateLineage` 或 `DecisionDelta`。

inner 只返回 `AttemptResult`：

- attempt 内发生了什么
- board 最终长什么样
- objection 更新了什么
- 当前 mode 和 degraded 状态是什么
- 下一步 review 需要什么输入

outer 再基于 `AttemptResult` 做 review 和 controller state promotion。

### 3. `discussion -> exploratory_snapshot`

`exploratory_snapshot` 不是“偷跑完整 attempt”的捷径。

它是低保证模式，必须显式声明：

- 没有跑完哪些 phase
- 哪些 objection 只是 hint，不是 disposition
- outer 不得把它当成已经完成 `critique / repair / judge` 的 full attempt

## Liveness Rules

- inner 只拥有 attempt 内 participant 生命周期
- outer 只拥有 attempt 之间 roster 生命周期
- inner 可以在 `AttemptResult.runtime_status` 中建议：
  - `retain`
  - `release`
  - `replay_required_next_time`
- outer 才能决定：
  - `continue_same_roster`
  - `revise_same_roster`
  - `retire_and_respawn`
  - `branch_new_direction`
  - `stop_and_summarize`

## Anti-Drift Rules

- 不要把 `discussion` 生命周期描述成 `campaign` 生命周期
- 不要把 `AttemptIntent` 直接当成 inner runtime packet
- 不要让 `ModeDisclosure` 同时承担 user-facing transparency 和 controller-only state
- 不要让 reviewer 建议语气直接写成 canonical queue status
- 不要让 `exploratory_snapshot` 和 `full_deliberation` 共用一套未区分 guarantee 的输出语义

## Quick Test

如果以下任何一个问题回答不出来，说明 ownership 还没定清：

1. 这个状态属于 `campaign`、`attempt`、`discussion` 还是 `exploratory_snapshot`？
2. 这个状态是谁最后解释？
3. 这个状态是否必须 survive replay / respawn / cross-session？
4. 这个状态是否可以直接被 outer 改写，还是只能由 inner 产出再被 outer promote？
