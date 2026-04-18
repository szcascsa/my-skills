---
name: subagent-discussion
description: This skill should be used when the user asks to "圆桌讨论", "自由讨论", "辩论", "专家会诊", "多角色评审", "继续这场讨论", "帮我深挖这个方案", or otherwise wants multiple subagents to iteratively generate, critique, repair, and judge ideas on one topic.
---

# Subagent Discussion

用一组 subagents 围绕同一议题进行结构化讨论，由主 agent 担任主持人，负责组局、投递、控场、筛选、收敛与交付。

这个 skill 只负责 **单次 attempt** 的 inner discussion engine，不负责跨 attempt 的 campaign 控制、branch 管理或 outer ledger。

## Core Principle

优先优化结果质量，而不是会话热闹度。

默认路径不是机械轮询，而是：

`frame -> blind_diverge -> critique -> repair -> judge -> verify`

其中：

- `objective_mode`
  决定要追求什么结果、按什么 rubric 判断、交付什么 artifact
- `discussion_format`
  决定主持人如何路由互动
- `phase_pipeline`
  决定当前处于生成、批评、修订、裁决还是核验

不要把三者混成一个概念。

## 何时使用

当用户需要以下能力时触发：

- 多个 subagents 或角色围绕同一议题生成候选、定向批评、修订和裁决
- 针对科研、产品、架构、策略问题进行一轮或多轮深入讨论
- 在同一 attempt 内保留 participant 记忆，并推进候选物演化
- 在已有 outer controller 的情况下，执行一次可审 deliberation

以下情况不要优先使用本 skill：

- 只需要查一个事实、命令、API 用法或单点答案
- 低不确定、明显模式的小修改
- 用户明确只要单个直接答案，不需要多视角探索

## 两种运行上下文

### 1. standalone

当没有 outer controller 时，最少只需要：

- `topic`
- `objective_mode`
- `final_artifact`

其余内容可以 lazy derive。

### 2. controller-invoked

当由 `iterative-deliberation-loop` 调用时，不要再靠自由文本隐式传状态。

必须把 outer state 收敛成一个 `AttemptPacket`，其 authoritative 定义见：

- [lifecycle-ownership.md](../iterative-deliberation-loop/references/lifecycle-ownership.md)
- [attempt-boundary.md](../iterative-deliberation-loop/references/attempt-boundary.md)

本 skill 在这种模式下只消费 `AttemptPacket`，并只返回 `AttemptResult`。

## Inner-Owned Objects

本 skill 默认拥有以下 inner 对象：

| 对象 | 作用 |
| --- | --- |
| `ObjectiveSpec` | 定义本次 attempt 内真正执行的目标、artifact 和停止条件 |
| `FactSheet` | 定义事实边界 |
| `QualityRubric` | 定义好结果的判断标准 |
| `DiscussionState` | 维护当前 phase、候选状态和 round budget |
| `ParticipantCard` | 定义参与者边界 |
| `CandidateArtifact` | 定义被生成、批评、修订的候选物 |
| `CritiqueMemo` | 定义结构化批评 |
| `SelectionBoard` | 定义 survivor / rejected / outlier |
| `ModeDisclosure` | 暴露当前 runtime transparency |
| `ModeratorSummary` | 汇总本次 attempt 的 decision-ready 输出 |

outer 不得在 inner 外面再维护一份近似的 discussion object。

## 输入契约

### standalone 最小输入

```yaml
topic: ""
objective_mode: idea_funnel | architecture_synthesis | pressure_test | decision_debate | sensemaking
final_artifact: ""
```

### controller-invoked 输入

直接消费 [attempt-boundary.md](../iterative-deliberation-loop/references/attempt-boundary.md) 中定义的 `AttemptPacket`。

特别注意：

- 不要把 `AttemptIntent` 原样当成 inner runtime packet
- 不要再额外注入不在 `AttemptPacket` 中的 controller 私有字段
- 若某字段经常需要传入，但不在 packet schema 中，要么升级进 `AttemptPacket`，要么删掉

## 输出契约

本 skill 每次 attempt 结束时都只返回一个 `AttemptResult`。

### `AttemptResult` 的强保证

- attempt identity 稳定
- 当前 mode 明确可见
- board snapshot 可审
- objection update 可提升
- artifact 可供 reviewer 和 outer controller 使用

### `AttemptResult` 的弱保证

若 `attempt_kind = exploratory_snapshot` 或 `completion_status = degraded`：

- 返回值仍然必须结构化
- 但 outer 不得把它当成已经完成 full `critique / repair / judge` 的 full attempt

## 运行模式

只使用一套 mode 词汇：

- `persistent`
- `replay`
- `degraded`

不要再并列维护另一套 `spokesperson / degraded_synthesis` 风格的 mode。

如果需要说明特殊降级原因，把它写进 `ModeDisclosure.degrade_event`，不要升级为新 mode。

runtime 细节见 [runtime-modes.md](references/runtime-modes.md)。

## Phase Pipeline

### `frame`

建立：

- `ObjectiveSpec`
- `FactSheet`
- `QualityRubric`
- 最小 stopping rule

### `blind_diverge`

让 `2-4` 位差异足够大的参与者独立生成候选。

默认要求：

- 每人 `1-2` 个候选
- 候选必须尽快显式写出 `killer_risk`
- 候选必须尽快显式写出 `fastest_test`
- 在 `idea_funnel` 下，首波优先追求可区分，而不是成熟

### `cluster_and_select`

主持人对候选做：

- 去重
- 聚类
- shortlist
- outlier 保留

进入第一次 `critique` 前，必须至少保留 `2` 个可区分候选，除非显式标记 `moderator-generated fallback`。

### `critique`

把 candidate 投给最相关的 `critic` / `red team`。

要求：

- 攻击 strongest stated thesis
- 不打 strawman
- 写清 `what would change my mind`

### `repair`

把结构化 objection 投回 `proposer` / `reviser`。

要求：

- 至少明确回应一个 prior objection
- 说明改了什么
- 说明哪些边界被收紧

### `judge`

主持人或指定 `judge` 排序、筛选、合并，形成推荐或 shortlist。

要求：

- 说明 `why not the others`
- 说明仍未关闭的 evidence gap

### `verify`

高风险或强依赖外部事实时，结束前单独做一轮事实 / 假设核验。

要求：

- 区分 `verified_fact`
- 区分 `inference`
- 区分 `speculation`

## Hard Guarantees

### 对 `full_deliberation`

输出最终 recommendation / shortlist 前，必须满足：

- 已有用户可见 artifact
- 已有 `SelectionBoard`
- 顶级 objection 至少标记为 `open / addressed / mitigated / closed`
- 已输出 `ModeDisclosure`

### 对 `exploratory_snapshot`

允许压缩流程，但必须显式声明：

- 跳过了哪些 phase
- 结果只属于 hypothesis generation 还是 partial pressure-test
- outer 不得把它视为 strong evidence of structural progress

## Liveness Rule

本 skill 只拥有 **一次 attempt 内** 的 participant 生命周期。

内层可以在 `AttemptResult.runtime_status` 中建议：

- `retain`
- `release`
- `replace`
- `replay_required_next_time`

但是否跨 attempt 继续复用 roster，由 outer controller 决定。

## Anti-Collapse Rules

- 前两轮默认不接受泛泛同意
- 每个 survivor candidate 至少经历一次真正的独立批评
- 至少保留一个高新颖、低置信的 outlier 到裁决前
- `judge` 不得只给漂亮总结，必须交付可比较 artifact
- `exploratory_snapshot` 不得伪装成 full attempt

## 面向用户的输出协议

默认优先输出 artifact，再补会话摘要。

建议结构：

```text
【主持】Mode Disclosure
- mode：persistent / replay / degraded
- content_origin：participant-originated / moderator-synthesized / mixed
- degrade_event：...
- synthesis_used：yes / no

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

更具体的 artifact 模板见 [artifact-templates.md](references/artifact-templates.md)。

## 实施提示

按需读取：

- [objective-modes.md](references/objective-modes.md)
- [discussion-formats.md](references/discussion-formats.md)
- [subagent-templates.md](references/subagent-templates.md)
- [runtime-modes.md](references/runtime-modes.md)
- [artifact-templates.md](references/artifact-templates.md)
- [attempt-boundary.md](../iterative-deliberation-loop/references/attempt-boundary.md)
- [lifecycle-ownership.md](../iterative-deliberation-loop/references/lifecycle-ownership.md)
