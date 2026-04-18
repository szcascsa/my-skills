# Artifact Templates

默认优先交付 artifact，而不是只交付会话摘要。

默认每个 artifact 前都应有一段轻量透明度提示：

```text
【主持】Mode Disclosure
- mode：persistent / replay / degraded
- content_origin：participant-originated / moderator-synthesized / mixed
- degrade_event：...
- synthesis_used：yes / no
```

如果当前 `attempt_kind = exploratory_snapshot` 或 `completion_status = degraded`，在 artifact 顶部追加：

```text
【主持】Attempt Guarantee
- attempt_kind：full_deliberation / exploratory_snapshot
- completion_status：completed / degraded / blocked
- skipped_phases：...
```

## 1. `idea portfolio`

适用：
- `idea_funnel`

```text
【主持】Idea Portfolio

1. Candidate: ...
- 核心想法：...
- 新意来自：...
- 为什么值得保留：...
- killer risk：...
- fastest test：...
- 当前置信度：...

2. Candidate: ...
...

【主持】Shortlist
- 推荐继续推进：...
- 推荐暂缓：...
- 还需要的证据：...
```

## 2. `option matrix`

适用：
- `architecture_synthesis`

```text
【主持】Option Matrix

| Option | 核心思路 | 最适合场景 | 主要风险 | 复杂度 | 可逆性 |
| --- | --- | --- | --- | --- | --- |
| A | ... | ... | ... | ... | ... |
| B | ... | ... | ... | ... | ... |
| C | ... | ... | ... | ... | ... |

【主持】Recommendation
- 当前推荐：...
- 推荐原因：...
- why not the others：...
- 哪些条件变化会改判：...
```

## 3. `decision memo`

适用：
- `decision_debate`
- 一般性决策收束

```text
【主持】Decision Memo
- 议题：...
- 当前判定：...
- 支撑理由：...
- 关键反对意见：...
- 尚未解决的证据缺口：...
- 建议下一步：...
```

## 4. `risk register`

适用：
- `pressure_test`

```text
【主持】Risk Register

| Risk | 严重度 | 触发条件 | 现有缓解 | 是否已被修补 |
| --- | --- | --- | --- | --- |
| ... | ... | ... | ... | ... |

【主持】Patched Version
- 修订后主张：...
- 仍然最脆弱的点：...
- 是否建议继续：...
```

## 5. `perspective map`

适用：
- `sensemaking`

```text
【主持】Perspective Map

| 视角 | 最关心什么 | 关键前提 | 最强反对意见 |
| --- | --- | --- | --- |
| ... | ... | ... | ... |

【主持】Structure Summary
- 当前真正的冲突：...
- 被忽视的前提：...
- 哪个问题最值得进入下一阶段：...
```

## 通用收尾块

不论 artifact 类型，结束时建议都补这一段：

```text
【主持】Open Questions
- ...

【主持】What would change the current recommendation
- ...

【主持】Objection Disposition
- open：...
- addressed：...
- mitigated：...
- closed：...

【主持】Next Step
- ...
```
