# Discussion Format Presets

这些模板定义的是“主持人如何路由互动”，不是“最终要产出什么”。

先选 `objective_mode`，再选 `discussion_format`。
不要把 format 当成目标本身。

## Global Rule

如果议题依赖外部事实、最新状态或可变化信息：
- 先发 `FactSheet`
- 若未核实，则显式写 `no fact verification performed`

所有 format 都必须服从当前 `phase_pipeline`：
- `frame`
- `blind_diverge`
- `critique`
- `repair`
- `judge`
- `verify`

动作标签与 canonical `move` 枚举保持一致；不要在某个 format 内私自引入新的内部动作类型。

## Format 不是 Objective

最常见的混淆是：
- `roundtable` 不是 brainstorm
- `free_discussion` 不是 ideation engine
- `debate` 不是 decision memo

更准确的关系是：
- format 决定互动方式
- objective mode 决定 artifact 和 rubric
- phase 决定当前这轮该生成、批评、修订还是裁决

## 1. `roundtable`

**目标**：暴露结构、定义裂缝、推进深问。  
**适合**：复杂问题的 framing、隐藏前提识别、跨学科议题拆解。

### 推荐人数

- 总阵容：`3-8`
- 每轮活跃发言者：`2-4`

### 推荐相位

- 最适合 `frame`
- 也适合 `critique`
- 通常不作为 `judge` 的最终形式

### 主持人路由策略

1. **首轮**
   - 如果 `N <= 6`，允许全员用一句定义或初始 framing 亮相
   - 如果 `N > 6`，只让核心代表先亮相，其他人待命

2. **第二轮起**
   - 每轮只激活最相关的 `2-4` 人
   - 至少指定一个“必须回应对象”
   - 至少保留一个旁路视角，防止讨论塌成二元对打

3. **主持人判断标准**
   - 哪条分歧最深
   - 哪个前提被大家默认接受但最值得拆
   - 哪个观点尚未被真正反驳

### 风险

- 很容易变成“大家都很有道理”的并排陈述
- 如果没有后续 `critique -> repair -> judge`，通常不会自然收敛出好方案

## 2. `free_discussion`

**目标**：快速放大异质连接和意外线索。  
**适合**：早期发散、旁路联想、概念重组。

### 推荐人数

- 总阵容：`3-10`
- 每个波次活跃发言者：`3-4`

### 推荐相位

- 最适合 `blind_diverge`
- 可短暂用于 `repair` 前的跨候选重组
- 不建议直接作为终局

### 主持人路由策略

1. **首波**
   - 向 `3-4` 位风格差异大的参与者并行投递同一问题
   - 尽量先做独立生成，再共享摘要

2. **第二波**
   - 主持人挑出最有新意、最冲突、最可组合的点，转投给新的参与者
   - 不要求逐点对打

3. **收束前**
   - 必须做去重、聚类和保留候选
   - 不能无限开波次

### 风险

- 最容易塌成低质量群聊
- 对科研 ideation，默认只能当发散相，不能代替筛选与批评

## 3. `debate`

**目标**：围绕可判别命题比较立场强弱。  
**适合**：命题真假、政策正反、方案路线之争。

### 推荐人数

- 最小：`2`
- 推荐：`4-8`
- 人数较多时，按阵营组织，而不是每个人都独立成方

### 推荐相位

- 最适合 `critique`
- 也可用于 `judge` 前的最后一轮对打

### 主持人路由策略

1. **命题澄清**
   - 先把命题写成一句能被反驳的话

2. **开篇陈词**
   - 正反方各派代表发言

3. **交叉质询**
   - 每轮把一方最强主张定向转投给对方
   - 要求对方先准确复述，再批判

4. **结辩**
   - 各方压缩成最强结论

### 风险

- 如果命题本身没压缩清楚，debate 只会制造姿态
- 如果强制所有人站队，很容易出现角色表演胜过真实推理

## 4. `expert_council`

**目标**：围绕真实决策给出诊断、选项、风险和建议。  
**适合**：产品方向、研究方案、工程决策、运营策略。

### 推荐人数

- 总阵容：`3-7`

### 推荐相位

- 最适合 `frame`
- 也适合 `judge`

### 主持人路由策略

1. **第一轮：独立诊断**
   - 每位专家先独立给出诊断和首选方案

2. **第二轮：互相指出风险**
   - 每个人必须指出别人方案的一条关键风险或遗漏

3. **第三轮：收敛**
   - 主持人要求给出排序、条件和适用边界

### 风险

- 如果没有“独立诊断”，很容易提前互相影响
- 如果没有结构化排序，最后会退化成“都可以，看场景”

## 5. `red_team`

**目标**：压测一个想法、计划、产品或论证，找出最容易失败的地方。  
**适合**：方案评审、上线前检查、论文论证压力测试、创业点子体检。

### 推荐人数

- 最小：`3`
- 推荐：`4-6`

### 推荐相位

- 最适合 `critique`
- 也适合 `repair` 后的二次压测

### 典型角色

- `proposer`
- `red`
- `blue`
- `operator`
- `judge`

### 主持人路由策略

1. 先让 `proposer` 把主张压缩成可攻击对象
2. 把主张投给 `red`，要求找最危险的失败路径
3. 把攻击点投给 `blue` 或 `proposer`，要求修正，而不是辩护
4. 再把修正版投给 `operator` 或 `judge`，看是否真的变强

### 风险

- 如果 `red` 攻击弱版本，就会浪费轮次
- 如果没有 `judge`，很容易陷入“红蓝互打但没人定版”

## 常用组合

| objective_mode | 推荐 format 组合 |
| --- | --- |
| `idea_funnel` | `free_discussion` or `roundtable` for `blind_diverge` + `red_team` or `expert_council` for `critique/judge` |
| `architecture_synthesis` | `expert_council` + `red_team` |
| `pressure_test` | `red_team`，必要时加 `expert_council` 做收敛 |
| `decision_debate` | `debate` + `judge` |
| `sensemaking` | `roundtable`，必要时短暂接 `expert_council` 收束 |

## Overlay Passes

以下两个 pass 不单独算 format，但常常值得加：

### `consistency_check`

在 `judge` 后，让 `1-3` 个独立参与者只做终局判断，不看彼此答案，再聚合。

适用：
- 高不确定结论
- 候选难分高下
- 需要降低偶然性

### `verification_pass`

在结束前，把最终判断中的事实、假设、推论分开检查。

适用：
- 高风险问题
- 强依赖最新事实
- 用户明确要“成熟方案”而不是“有趣讨论”
