# my-skills

个人 Codex / subagent 资产仓库。

当前仓库按两层组织：

- `skills/`
  存放可直接安装或复制的 skill 包
- `agents/`
  存放独立的 subagent 文件

## Included Skills

- `skills/subagent-discussion`
  单次多角色深度讨论引擎，负责 `frame -> blind diverge -> critique -> repair -> judge -> verify`
- `skills/iterative-deliberation-loop`
  跨 attempt 的 outer controller，负责 review loop、branch control、objection lifecycle、cross-session resume
- `skills/root-cause-roundtable`
  多个 `root-cause-architect` 的根因会诊与收敛引擎，负责 `blind isolation -> clustering -> cross-examination -> forced convergence -> exhaustive convergence loop`

## Included Agents

- `agents/claude-opus-style.toml`
  独立的 deliberative 风格 subagent 文件
- `agents/root-cause-architect.toml`
  极限根因深挖 subagent，默认 `xhigh` 推理强度，专注主导根因、控制性约束与高杠杆修复点

## Skill-Local Agent Files

- `skills/iterative-deliberation-loop/agents/openai.yaml`
  `iterative-deliberation-loop` skill 自带的配套配置文件
- `skills/root-cause-roundtable/agents/openai.yaml`
  `root-cause-roundtable` skill 自带的配套配置文件

## Relationship

- `subagent-discussion` 解决“单轮深度讨论怎么跑”
- `iterative-deliberation-loop` 解决“多轮讨论如何审阅、继续、分支、重组和恢复”
- `root-cause-architect` 提供“只追主因、不接受表层解释”的单体分析能力
- `root-cause-roundtable` 把多个 `root-cause-architect` 组织成标准/穷尽两种模式的根因收敛流程

## Suggested Local Layout

实际安装到 Codex 时，skill 仍建议保留各自完整目录：

```text
~/.codex/skills/
  subagent-discussion/
  iterative-deliberation-loop/
```

根目录 `agents/` 只放独立 subagent 文件。
skill 自己的配套文件继续留在对应 skill 目录内部。
