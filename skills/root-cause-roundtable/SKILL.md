---
name: root-cause-roundtable
description: 用多个 root-cause-architect 对同一问题做全方位、多角度、多轮次的根因会诊与结论收敛。Use when the user asks for 根因会诊、深挖讨论、多视角根因分析、多轮收敛、多个 root-cause-architect 一起讨论、全方位找问题根源、架构级追因，或希望对同一问题先独立深挖、再交叉质询、最后收敛出最终判断。When the user says `exhaustively`、穷尽式、一直讨论到收敛、不要三轮就停，switch to exhaustive convergence mode instead of the standard capped-round flow.
---

# Root Cause Roundtable

## Overview

组织多个 `root-cause-architect` 围绕同一议题做结构化深挖。目标不是堆更多观点，而是在多轮独立分析、交叉质询和强制收敛后，锁定单一主导根因，或得到有排序的根因簇与明确的证据缺口。默认提供 `standard` 与 `exhaustive` 两种运行模式。

## Preconditions

- 确认 `root-cause-architect` 已存在于 `C:\Users\17739\.codex\agents\root-cause-architect.toml`。
- 如果调用时返回 `unknown agent_type 'root-cause-architect'`，不要静默降级；直接告诉用户当前会话未加载自定义 agent，需要重开 Codex 或新开线程后再试。
- 议题依赖最新外部事实、价格、政策、版本、线上状态时，先核实事实，再把核实后的 `FactSheet` 传给各 subagent。
- 只在问题确实存在多种 plausible explanation、且表面现象容易误导时使用本 skill。单点事实查询、低不确定小改动、明显模式问题，不要拉多 agent 讨论。
- 在启动前读取 [lenses-and-rounds.md](references/lenses-and-rounds.md) 选择不重叠的深挖透镜。

## Run Modes

默认不是“越多人越好”，而是“足够多的正交透镜，足够少的冗余”。

### `standard`

- 用户没有特别要求时使用
- 目标是快速形成高质量收敛，而不是无限扩轮
- 默认按 `round 1 blind isolation -> round 2 cross-examination -> round 3 forced convergence` 推进
- round 3 之后只有在“新增证据很可能改变胜负”时才追加一轮

### `exhaustive`

- 用户明确说 `exhaustively`、穷尽式、一直讨论到收敛、不要三轮就停时使用
- 不采用固定三轮停止规则；前三轮只是起始框架，不是上限
- round 3 之后进入 `exhaustive convergence loop`
- 不允许仅因为“三轮已完成”或“信息增量变低”而停止
- 只有在多数 agent 已经收敛，或用户主动停止，或遇到硬性证据阻塞时才允许收束

### Majority Convergence Rule

`exhaustive` 模式下，主持人必须显式计算收敛度。

默认把“多数 agent 已收敛”定义为：

- 至少 `2/3` 的 active agents 指向同一 `dominant_root_cause`
- 或至少 `2/3` 的 active agents 指向同一等价根因簇，只是表述不同
- 且最新一轮没有出现新的非冗余候选家族

如果仍有 dissenting agents，主持人必须判断他们是在提出：

- 新的候选家族
- 旧候选的更强版本
- 还是纯粹重复已被驳回的路径

只有前两者才值得继续扩轮。

## Default Shape

复杂度与预算建议：

- `low`: `3` 个 agent，`2` 轮
- `medium`: `4-5` 个 agent，`3` 轮
- `high`: `5-6` 个 agent，`3-4` 轮

`standard` 模式默认停止条件：

- 已锁定单一主导根因，并给出完整因果链
- 或已形成 `top-3` 根因簇排序，同时明确哪些未知量会改变排序
- 或连续两轮新增信息过低，继续讨论只会重复

`exhaustive` 模式默认停止条件：

- 已满足 `majority convergence`
- 或用户明确要求停止
- 或所有剩余分歧都已被压缩为“需要新增外部证据才能继续判断”的证据阻塞，而不是推理层分歧

## Launch Rules

优先并行拉起多个同类型 `root-cause-architect`，但每个 agent 必须绑定不同透镜，不允许“同一句话换个说法”式重复。

默认推荐透镜组合：

- `boundary-owner`
- `execution-chain`
- `state-invariant`
- `dependency-ops`
- `counterfactual-pruner`
- `leverage-fix`

如果问题更像长期组织性堵点，而不是纯技术故障，把 `leverage-fix` 替换成 `incentive-process`。

优先复用同一批 persistent subagents 做多轮推进：

- round 1 后不要立刻关闭
- 后续用 `send_input` 继续追问和交叉质询
- 只有在下一步被结果阻塞时才 `wait_agent`

如果环境不支持持久复用，则在新一轮 prompt 中附带上一轮压缩版结论，而不是把整段历史原样回灌。

## Workflow

### 1. Frame the case

先写出一个紧凑 case brief，至少包含：

- 目标结果
- 已观察到的表面症状
- 作用边界
- 已知事实与未知量
- 当前最强的几个竞争解释
- 收敛标准

不要一上来就把“怀疑原因”当结论。主持人先把症状、事实、假设拆开。

### 2. Round 1: Blind isolation

并行启动多个 `root-cause-architect`，每个绑定一个独立透镜。所有 agent 共享同一 case brief，但不共享彼此答案。

要求每个 agent 返回：

- `dominant_cause`
- `causal_chain`
- `rejected_surface_explanations`
- `disconfirming_evidence_needed`
- `highest_leverage_fix`
- `confidence`

### 3. Moderator clustering

主持人将 round 1 结果聚类成候选板：

- 合并同根不同表述
- 保留 `2-4` 个最强候选
- 至少保留 `1` 个高新颖度 outlier，防止过早塌缩成伪共识

### 4. Round 2: Cross-examination

把候选板回投给各 agent，但这一次要让他们攻击彼此最强版本，而不是继续自说自话。

每个 agent 至少回答：

- 哪个竞争解释最像真的，为什么
- 它为什么仍然不如自己的判断
- 什么证据会迫使自己改判
- 当前结论最薄弱的一环在哪里

### 5. Round 3: Forced convergence

让每个 agent 在读过交叉质询后重新排序候选，并强制回答这个问题：

如果现在只能修一个因素，哪一个因素最有可能改变结果，为什么不是第二名。

如果是 `standard` 模式，且 round 3 后仍然分裂：

- 只在“新增证据有可能改变胜负”时追加一轮
- 否则停止扩张，直接输出“当前无法单点收敛”的理由和证据缺口

如果是 `exhaustive` 模式，round 3 只是第一次收敛检查，不得把它当结束条件。

### 6. Exhaustive convergence loop

仅在 `exhaustive` 模式下启用。

round 3 之后，主持人进入循环，直到满足 `majority convergence`。

每一轮都必须只做一件事：缩小剩余分歧。

循环步骤：

1. 主持人公布当前 `convergence board`
   - 当前第一名根因或根因簇
   - 支持该结论的 agent 数量与比例
   - 仍在 dissent 的 agent
   - 本轮要消灭的那一个核心分歧

2. 只激活最相关的 `2-4` 位 agent
   - 一位代表当前主流结论
   - 一位或多位代表最强 dissent
   - 必要时加入一位 `counterfactual-pruner` 或 `leverage-fix`

3. 强制每位 active agent 回答：
   - 我是否加入当前主流结论；如果不加入，差异到底在哪里
   - 当前主流结论最可能错在哪一步
   - 什么证据会让我并入主流
   - 我是否在提出真正新信息，而不是重复旧 objection

4. 主持人重新计算 `convergence ratio`
   - 如果达到 `majority convergence`，允许结束
   - 如果没有达到，则继续下一轮

`exhaustive` 模式下，不允许用“讨论有点重复了”作为结束理由。必须明确说明为什么当前剩余分歧仍值得继续，或为什么它已经退化成纯证据阻塞。

### 7. Final synthesis

主持人最终交付的不是对话摘要，而是 `ConvergenceMemo`：

- `dominant_root_cause`
- `causal_chain`
- `ranked_contributing_factors`
- `rejected_explanations`
- `key_evidence_gaps`
- `highest_leverage_fix`
- `next_proving_step`
- `run_mode`
- `rounds_executed`
- `convergence_ratio`

## Discussion Discipline

- 不接受 laundry list；默认要求每个 agent 只押注一个主导因素
- 不接受“都重要”；必须区分主因、贡献因素、噪声
- 不接受表象修补；必须说明为什么问题会反复出现
- 不接受只讲现象；必须向下追到控制性约束、架构级堵点或最小致因单元
- 不接受空泛反驳；批评必须附带“什么证据会让我改判”
- `standard` 模式不接受无限轮次；`exhaustively` 模式允许继续，但每多一轮都要明确它在消灭哪一类剩余分歧

## Pressure-Test Variant

如果用户已经给出一个怀疑中的根因，不要再做完整发散。改为：

- `frame`
- `challenge suspected cause`
- `find stronger alternative`
- `judge`

目标是判断这个根因是不是“真的最主导”，而不是替用户把原假设包装得更漂亮。

## Output Contract

最终输出优先使用以下结构：

```text
Root-Cause Board
- Dominant root cause:
- Causal chain:
- Ranked contributing factors:
- Rejected explanations:
- Evidence still missing:
- Highest-leverage fix:
- Next proving step:

Round Notes
- Run mode:
- Rounds executed:
- Round 1 divergence:
- Round 2 strongest attack:
- Round 3 convergence state:
- Exhaustive loop state:
- Final convergence ratio:
```

如果结论无法完全收敛，也必须明确写出：

- 当前第一名与第二名分别是什么
- 哪个未知量可能翻转排序
- 为什么现在不该继续空转讨论

## Implementation Notes

- 透镜卡和每一轮的 prompt 骨架见 [lenses-and-rounds.md](references/lenses-and-rounds.md)。
- 主线程负责主持、聚类、挑选、收敛；不要把综合工作再次外包成新的泛化 agent。
- 如果同一轮中两个 agent 的视角明显重叠，下一轮应主动换透镜，而不是保留冗余席位。
- 如果用户明确说 `exhaustively`，主持人必须把 `run_mode` 标成 `exhaustive`，并显式汇报每轮收敛比例。
