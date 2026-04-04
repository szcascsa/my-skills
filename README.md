# my-skills

个人 Codex / subagent 资产仓库。

当前仓库按两层组织：

- `skills/`
  存放可直接安装或复制的 skill 包
- `agents/`
  存放与这些 skill 配套使用的 agent 配置

## Included Skills

- `skills/subagent-discussion`
  单次多角色深度讨论引擎，负责 `frame -> blind diverge -> critique -> repair -> judge -> verify`
- `skills/iterative-deliberation-loop`
  跨 attempt 的 outer controller，负责 review loop、branch control、objection lifecycle、cross-session resume

## Included Agents

- `agents/subagent-discussion/claude-opus-style.toml`
  配套 `subagent-discussion` 的 deliberative 风格 agent
- `agents/iterative-deliberation-loop/openai.yaml`
  配套 `iterative-deliberation-loop` 的 outer review / control agent

## Relationship

- `subagent-discussion` 解决“单轮深度讨论怎么跑”
- `iterative-deliberation-loop` 解决“多轮讨论如何审阅、继续、分支、重组和恢复”

## Suggested Local Layout

实际安装到 Codex 时，skill 仍建议保留各自完整目录：

```text
~/.codex/skills/
  subagent-discussion/
  iterative-deliberation-loop/
```

本仓库把 `agents/` 独立出来，主要是为了让远程仓库结构更清晰，便于管理与复用。
