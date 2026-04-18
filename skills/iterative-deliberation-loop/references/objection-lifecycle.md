# Objection Lifecycle

`DecisiveObjectionQueue` 只跟踪会改变走向的 objection family。

不要把所有 open issue 都塞进来。

## Canonical Status Set

统一使用以下状态：

- `open`
- `addressed`
- `mitigated`
- `closed`
- `accepted_risk`
- `reopened`
- `escalated`

不要再并列维护：

- `accepted / mitigated / unresolved`
- `still_open`

这类 reviewer 或 artifact 口吻不能充当 canonical queue status。

## 何时进入 Queue

只有满足以下任一条件的问题，才值得进 queue：

- 会改变 `continue / revise / branch / respawn / stop` 的选择
- 会推翻当前 candidate family 的 viability
- 会改变 branch 排序
- 会触发 `VerificationGate`

普通备注、措辞问题、局部优化建议，不要进 queue。

## 最小字段

```text
objection_id:
target_candidate_id:
status:
introduced_by_attempt:
evidence_needed:
```

必要时可临时补：

- `family_label`
- `priority`
- `last_touched_attempt`
- `close_basis`

## Authority Split

### inner 可以产出

- `open`
- `addressed`
- `mitigated`
- `closed`

### outer controller 可以追加

- `accepted_risk`
- `reopened`
- `escalated`

reviewer 不直接改 canonical state，只提供建议。最终 queue status 由 controller 写入。

## Lifecycle Operations

### `introduce`

首次把一个 route-changing 问题放入 queue，默认状态写成 `open`。

### `promote`

把原本普通的 objection update 升级成 decisive objection。

适用：

- 新证据表明它会改变下一控制决策
- 它在后续成为 killer risk

### `merge`

把两个本质相同的 objection family 合并。

要求：

- 保留一个 canonical `objection_id`
- 在 `state_changes` 中写清被并入的是谁

### 写成 `addressed`

某个 attempt 明确尝试回应这个 objection 时，把状态写成 `addressed`。

注意：

- `addressed` 不等于 `mitigated`
- `mitigated` 不等于 `closed`

### 写成 `mitigated`

objection 仍然存在，但其威力已明显下降，不再同等强烈地阻塞当前路线时，把状态写成 `mitigated`。

### 写成 `closed`

reviewer 或验证结果确认该 objection 已不再改变走向时，把状态写成 `closed`。

### 写成 `accepted_risk`

objection 仍然成立，但 controller 显式接受该风险并继续时，把状态写成 `accepted_risk`。

### 写成 `reopened`

已 `closed` 或 `mitigated` 的 objection 因 later evidence、thesis drift 或 branch drift 重新变得 load-bearing 时，把状态写成 `reopened`。

### 写成 `escalated`

同一 objection family 在同一 branch 上连续 `2` 次 attempt 没有真实 state delta 时，把状态升级为 `escalated`。

一旦 `escalated`，不要再把它当普通 revise 处理。

## Escalation Rule

出现以下任一情况时，把 objection 升级处理：

- 连续两轮只被 `addressed`
- 对它的回应需要新 thesis family
- 它要求先过 `VerificationGate`
- reviewer 明确指出当前 roster 无法解决它

升级后的常见动作：

- `branch_new_direction`
- `retire_and_respawn`
- `stop_and_summarize`

## Queue Hygiene

- 不要追踪所有小问题
- 不要无证据地把状态写成 `closed`
- 不要把 `addressed` 伪装成 `closed`
- 不要把 `mitigated` 伪装成 `closed`
- 不要静默丢弃后来升级为 killer-risk 的 latent objection
