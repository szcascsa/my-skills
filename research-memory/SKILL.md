---
name: research-memory
description: Use when 用户在多轮科研协作中积累了过多想法、实验和项目背景，担心模型忘记上下文，或需要把 research context、hypotheses、experiment logs、decisions 和 session handoffs 外置为可检索的长期记忆文件，用于 literature review、project planning、experiment execution、result analysis 或长期协作。
---

# 科研记忆

## 概述

把科研项目变成一个落盘的长期记忆系统。对话继续负责推理，项目背景、点子、实验、决策和交接摘要则写入文件，供后续会话按需检索。

## 核心原则

不要把长对话历史当作事实源。

- 对话负责当前推理。
- 文件负责稳定记忆。
- 每次新讨论前只回填当前问题真正需要的记忆片段。

## 快速开始

在当前工作区建立一个科研记忆根目录。默认使用 `research-memory/`，除非项目里已经有更合适的位置，例如 `docs/research-memory/`。

先建立最小目录结构：

```text
research-memory/
  project_context.md
  index.md
  ideas/
  experiments/
  decisions/
  sessions/
  papers/
```

然后执行：

1. 在 `project_context.md` 中写清项目的稳定事实。
2. 用 `index.md` 做入口和目录。
3. 每个 idea、experiment、decision、session handoff 各自单独成文件。
4. 每次新会话开始前，先读 `project_context.md`、最近几份 handoff，以及当前主题相关文件。

## 记忆分层

| 层级 | 作用 | 更新频率 |
| --- | --- | --- |
| `project_context.md` | 项目的稳定事实、术语、数据集、指标、代码入口、约束 | 只有项目现实变化时才更新 |
| `ideas/` | 假设、试验方向、开放问题 | 每出现一个新点子就更新 |
| `experiments/` | 改了什么、怎么跑的、结果如何 | 每次实验都记录 |
| `decisions/` | 为什么选择或放弃某条路径 | 每次真实取舍落定时更新 |
| `sessions/` | 给下一次会话的交接摘要 | 每次有实质推进的会话结束时更新 |
| `papers/` | 针对具体论文或 related work 主题的笔记 | 阅读和综合过程中更新 |

## 操作循环

### 1. 先判断这轮讨论属于哪种记忆对象

在深入回答前，先把当前请求映射到正确的记忆载体：

- 新假设或新方向：创建或更新一个 idea 文件。
- 拟做或已完成的实验：创建或更新一个 experiment 文件。
- 取舍或结论：创建或更新一个 decision 文件。
- 会话收尾：写一个 session handoff。
- 反复出现的项目背景事实：沉淀进 `project_context.md`。

### 2. 在信息漂移前先落盘

只要讨论产出了“下次还要用”的信息，就立刻写入文件，不要把它留在聊天记录里等着遗忘。

推荐做法：

- 一个文件只承载一个核心对象
- 使用稳定编号，例如 `IDEA-001.md`、`DEC-003.md`、`EXP-2026-04-02-01.md`
- 用显式状态字段，例如 `inbox`、`queued`、`running`、`validated`、`rejected`
- 在相关文件之间建立交叉链接

### 3. 推理前先检索

新会话开始时，或话题切换后，只加载真正相关的文件：

1. `project_context.md`
2. `sessions/` 中最近 1 到 3 份文件
3. 当前主题相关的 idea、experiment、decision 或 paper 文件

不要把整个记忆库都塞进上下文。只做窄检索。

### 4. 用 handoff 收尾

凡是改变了项目状态的会话，都在 `sessions/` 中留下一个 handoff，至少包含：

- 改了什么
- 学到了什么
- 还有什么没解决
- 下次优先读什么

## 检索规则

用下面的规则控制上下文体积：

| 场景 | 应加载内容 |
| --- | --- |
| 同一项目的新会话 | `project_context.md` + 最近的 session handoff |
| 讨论某一个具体点子 | 该 idea 文件 + 相关的 decision 或 experiment |
| 分析失败实验 | 对应的 experiment 文件 + 相关代码或配置引用 |
| 规划下一步 | 当前活跃的 ideas + 最近 decisions + 最新 handoff |
| 撰写论文某一节 | 相关 paper notes + 支撑实验 + 项目背景 |

## 文件卫生

- 稳定事实放在 `project_context.md`，不要埋在 session 日志里。
- 不要把 idea、experiment 和 decision 混在一个滚动大笔记里。
- 只要新增文件改变了导航面，就同步更新 `index.md`。
- 某个 idea 被证伪时，不要删历史，保留文件并标记为 rejected。
- 某个 decision 被替代时，新建一个 decision 文件并做好双向链接。

## 推荐命名

使用可排序、可复用的命名方式：

- Ideas: `IDEA-001-short-title.md`
- Experiments: `EXP-YYYY-MM-DD-01-short-title.md`
- Decisions: `DEC-001-short-title.md`
- Sessions: `YYYY-MM-DD-topic.md`
- Papers: `PAPER-short-citation.md`

## 模板

需要建文件时，直接参考 [references/templates.md](references/templates.md) 中的模板：

- `project_context.md`
- `index.md`
- idea 记录
- experiment 记录
- decision 记录
- session handoff

## 常见错误

| 错误 | 修正方式 |
| --- | --- |
| 什么都留在聊天里 | 只要信息可复用，就立刻写入文件 |
| 只有叙述，没有结构 | 补上状态、证据、下一步、关联文件 |
| 新会话直接开聊 | 先检索项目背景和最近 handoff |
| 被否掉的 idea 直接删除 | 保留文件，并明确记录否决原因 |
| 用一个超大笔记本记所有东西 | 按对象类型和主题拆文件 |

## 完成标准

当满足下面这些条件时，科研记忆系统才算进入可用状态：

- `project_context.md` 已存在，并且反映当前项目现实
- 每个活跃 idea 都有独立文件
- 每个关键 experiment 都记录了配置和结果
- 重要取舍都有 decision 记录
- 最新 handoff 能让下一位 agent 直接继续
- `index.md` 能让新的 agent 快速找到正确的记忆切片
