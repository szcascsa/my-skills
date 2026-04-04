# Adversarial Scenarios

用这些场景压测 skill，检查它是否真的比旧版本更稳。

## 1. Targeted Local Repair

### 场景

同一 branch 下，只想关闭一个很窄的 objection，例如“缺少 rollback plan”。

### 预期

- `AttemptIntent` 明确写这个 objection
- reviewer 判 `structural_progress = local`
- 动作倾向 `continue_same_roster` 或 `revise_same_roster`
- 不应误判成 `no structural progress`

## 2. Branch Smuggling

### 场景

主持人说“只是 revise”，但新稿其实换了 thesis family 或 success condition。

### 预期

- `BranchGate` 被触发
- 主持人必须解释 `why not branch`
- 如果解释不成立，动作应升级为 `branch_new_direction`

## 3. Single-Branch High-Risk

### 场景

只有一条 active branch，但关键证据缺口会影响 `revise` vs `respawn`。

### 预期

- reviewer 明确标 `verification_needed = required_before_next_control_decision`
- 先过 `VerificationGate`
- 不应因为没有 branch ordering 可比，就放过证据缺口

## 4. Latent Objection Promotion

### 场景

上一轮一个普通 open issue，这一轮被证明是 killer risk。

### 预期

- 主持人或 reviewer 执行 `promote`
- 问题进入 `DecisiveObjectionQueue`
- 之后的 decision 必须显式考虑它，而不是假装它一直只是备注

## 5. Escalated Stagnation

### 场景

同一 objection family 在同一 branch 上连续两轮只是 `addressed`。

### 预期

- objection 被标成 `escalated`
- 不应再无条件选择 `continue_same_roster`
- 主持人应在 `branch_new_direction / retire_and_respawn / stop_and_summarize` 中做实质裁决
