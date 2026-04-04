# Objection Lifecycle

`DecisiveObjectionQueue` 只跟踪会改变走向的 objection family。

不要把所有 open issue 都塞进来。

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

## Lifecycle Operations

### `introduce`

首次把一个 route-changing 问题放入 queue。

谁可以做：
- reviewer
- 主持人

### `promote`

把原本普通的 open issue 升级成 decisive objection。

适用：
- 新证据表明它会改变下一控制决策
- 它在后续成为 killer risk

### `merge`

把两个本质相同的 objection family 合并。

要求：
- 保留一个 canonical `objection_id`
- 在 `state_changes` 中写清被并入的是谁

### `address`

某个 attempt 明确尝试回应这个 objection。

注意：
- `addressed` 不等于 `closed`
- 只有“本轮确实试图处理它”，才允许写成 `addressed`

### `close`

reviewer 或验证结果确认该 objection 已不再改变走向。

谁可以做：
- reviewer
- 主持人，但必须引用 reviewer verdict 或 verification result

### `accept_risk`

objection 仍然成立，但主持人显式接受该风险并继续。

适用：
- 风险可接受
- 当前目标不是消灭它，而是把它透明地带进最终建议

### `reopen`

已关闭 objection 因 later evidence、thesis drift 或 branch drift 重新变得 load-bearing。

### `demote`

某个 objection 不再 route-changing，降回普通 open question。

要求：
- 在 `state_changes` 里写 `demoted_to_note`
- 不要默默从 queue 删除

### `escalate`

同一 objection family 在同一 branch 上连续 `2` 次 attempt 没有真实 state delta 时，升级为 `escalated`。

一旦 `escalated`，不要再把它当普通 revise 处理。

## Recommended Status Set

默认使用这些 status：
- `open`
- `addressed`
- `closed`
- `accepted_risk`
- `reopened`
- `escalated`

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
- 不要无证据地 `close`
- 不要把 `addressed` 伪装成 `closed`
- 不要静默丢弃后来升级为 killer-risk 的 latent objection
