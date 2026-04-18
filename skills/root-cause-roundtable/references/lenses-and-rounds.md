# Lenses and Rounds

在启动 `root-cause-roundtable` 前，先从这里挑选互相正交的透镜。目标不是覆盖一切，而是让每个 `root-cause-architect` 真正盯住不同的控制变量。

## Run Mode Switch

### `standard`

- 默认模式
- 适合需要高质量但有限轮次的根因收敛
- round 3 通常就是第一次也是最后一次强制收敛

### `exhaustive`

- 当用户说 `exhaustively`、穷尽式、一直讨论到收敛、不要三轮就停时启用
- round 3 只是第一次收敛检测，之后继续跑收敛循环
- 只有多数 agent 已经并入同一主因或同一等价根因簇时才允许停
- 默认把“多数”定义为 `>= 2/3` 的 active agents

## Lens Cards

### `boundary-owner`

- 关注边界划分、责任归属、模块耦合、跨层泄漏、错误抽象
- 适合发现“问题一直修不干净，因为责任边界本身错了”
- 交付重点：`dominant choke point`, `ownership failure`, `boundary-level fix`

### `execution-chain`

- 关注触发路径、控制流、时序、竞态、异常传播、从输入到症状的链路
- 适合发现“真正出错的位置不在报错位置，而在更早的触发环节”
- 交付重点：`first failure point`, `propagation chain`, `ordering-sensitive cause`

### `state-invariant`

- 关注数据状态、不变量、缓存、持久化、重复写入、读写时机
- 适合发现“症状很多，但都来自一个被破坏的不变量”
- 交付重点：`broken invariant`, `state corruption path`, `minimal invariant restore`

### `dependency-ops`

- 关注配置、环境差异、第三方依赖、超时、重试、降级、发布路径
- 适合发现“代码表面没问题，真正的根因在环境或依赖交互”
- 交付重点：`environmental trigger`, `dependency choke point`, `operational fix`

### `counterfactual-pruner`

- 关注反事实检验：去掉哪个因素后，最多症状会一起消失
- 专门负责砍掉伪根因、二级原因和好看但不主导的解释
- 交付重点：`single strongest cause`, `false trails`, `why not the others`

### `leverage-fix`

- 关注最小改动、高杠杆、复发预防、优雅修复
- 适合在后半程判断“即使多个因素都成立，先修哪个最值”
- 交付重点：`highest leverage intervention`, `recurrence prevention`, `cost-to-impact`

### `incentive-process`

- 关注组织流程、接口约束、评审机制、激励扭曲、长期回归原因
- 适合发现“技术问题的根，实际埋在流程或协作结构里”
- 交付重点：`systemic persistence reason`, `process choke point`, `anti-regression change`

## Round Templates

以下模板不是逐字照抄，而是每轮的最小约束。

### Round 1: Blind Isolation

给每个 `root-cause-architect` 的核心要求：

- 只用分配给你的透镜分析
- 不要罗列一串可能性，先给出一个 `dominant_cause`
- 写出从根因到表象的因果链
- 指出最容易误导人的表面解释
- 说明什么证据会打脸你的判断

建议骨架：

```text
You are one seat in a root-cause roundtable.
Lens: {lens}
Analyze the same case only through this lens.
Do not produce a laundry list.
Return:
- dominant_cause
- causal_chain
- rejected_surface_explanations
- disconfirming_evidence_needed
- highest_leverage_fix
- confidence
```

### Round 2: Cross-Examination

给每个 agent 的核心要求：

- 攻击最强竞争解释，不要打 strawman
- 指出自己的结论最薄弱的一环
- 明确什么新证据会导致改判

建议骨架：

```text
Read the rival candidate board.
Attack the strongest competing explanation, not the weakest one.
Then re-evaluate your own thesis.
Return:
- strongest_rival
- why_it_is_strong
- why_it_still_loses_or_wins
- weak_point_in_my_case
- evidence_that_would_change_my_mind
```

### Round 3: Forced Convergence

给每个 agent 的核心要求：

- 在所有候选中重新排序
- 如果只能修一个因素，必须选一个
- 如果仍无法单点收敛，明确指出卡住判断的未知量

建议骨架：

```text
Re-rank the surviving candidates after cross-examination.
If only one factor could be fixed now, choose it.
Return:
- final_ranking
- dominant_root_cause
- why_not_second_place
- unknown_that_could_flip_the_ranking
- next_proving_step
```

### Rounds 4+: Exhaustive Convergence Loop

仅在 `exhaustive` 模式下使用。

给每个 active agent 的核心要求：

- 判断自己是否加入当前主流结论
- 如果不加入，必须指出和主流的最小分歧点，而不是重讲整套理论
- 说明自己这次是否真的提供了新信息
- 明确什么证据会让自己收敛

建议骨架：

```text
Current leader cluster: {leader_cluster}
Current convergence ratio: {ratio}
Your task is to reduce the remaining disagreement, not restart the debate.
Return:
- do_i_join_the_leader_cluster
- if_not_joining_minimal_difference
- strongest_remaining_objection
- genuinely_new_information_this_round
- evidence_needed_to_converge
```

## Moderator Rules

- 同一轮不要给两个 agent 几乎相同的透镜
- round 1 先盲分析，禁止彼此看答案
- round 2 才允许交叉攻击
- round 3 必须强制排序，不能停在“都有道理”
- `standard` 模式下，如果新增信息不足，可以停止
- `exhaustive` 模式下，如果尚未达到多数收敛，不得仅因“信息不足”停止；必须继续压缩分歧，或明确宣布进入证据阻塞状态
