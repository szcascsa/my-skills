# 科研记忆模板

## project_context.md

```markdown
# 项目背景

## 目标
- 用一句话说明研究目标。

## 核心问题
- 问题 1
- 问题 2

## 范围
- 包含：
- 不包含：

## 数据
- 数据集：
- 版本：
- 划分方式：
- 预处理：

## 指标
- 主指标：
- 次指标：

## 代码面
- 主要入口：
- 关键目录：
- 常用命令：

## 约束
- 算力：
- 时间：
- 可复现性：
- 外部依赖：

## 当前工作假设
- 假设 1
- 假设 2

## 已知风险
- 风险：
- 缓解方式：
```

## index.md

```markdown
# 科研记忆索引

## 项目
- [项目背景](project_context.md)

## 当前活跃的 Ideas
- [IDEA-001-short-title](ideas/IDEA-001-short-title.md)

## 当前活跃的 Experiments
- [EXP-2026-04-02-01-short-title](experiments/EXP-2026-04-02-01-short-title.md)

## Decisions
- [DEC-001-short-title](decisions/DEC-001-short-title.md)

## 最近 Sessions
- [2026-04-02-main-thread](sessions/2026-04-02-main-thread.md)

## Papers
- [PAPER-short-citation](papers/PAPER-short-citation.md)
```

## Idea 模板

```markdown
# IDEA-001: 简短标题

## 状态
- 收件箱 inbox | 讨论中 discussing | 已排队 queued | 进行中 running | 已验证 validated | 已否决 rejected

## 问题
- 这次到底想验证什么？

## 假设
- 我们认为可能有效的机制是什么？

## 为什么值得测试
- 预期收益
- 现有方法为什么可能不够

## 预期信号
- 什么结果会支持这个假设？
- 什么结果会否定这个假设？

## 成本与依赖
- 算力
- 数据
- 代码依赖
- 外部论文或工具

## 风险
- 风险 1
- 风险 2

## 关联文件
- Experiments：
- Decisions：
- Papers：

## 下一步
- 下一条明确动作
```

## Experiment 模板

```markdown
# EXP-YYYY-MM-DD-01: 简短标题

## 状态
- 计划 planned | 运行中 running | 已完成 completed | 失败 failed | 已失效 invalidated

## 关联 Idea
- IDEA-001-short-title

## 假设
- 这次实验要测试的命题

## 变更集合
- 改了哪些代码
- 改了哪些配置
- 改了哪些数据

## 环境
- Commit：
- Branch：
- 机器：
- Seed：

## 过程
- 精确命令
- 评估流程

## 结果
- 主要指标
- 定性观察

## 解释
- 这些结果意味着什么
- 是否支持原始假设

## 下一步
- 后续动作
```

## Decision 模板

```markdown
# DEC-001: 简短标题

## 状态
- 生效 active | 已替代 superseded

## 决策
- 最终决定了什么

## 背景
- 是什么问题迫使我们做这个决策？

## 候选方案
- 方案 A
- 方案 B
- 方案 C

## 为什么选这个
- 证据
- 取舍
- 可逆性

## 后果
- 现在会改变什么
- 还残留哪些风险

## 关联文件
- Ideas：
- Experiments：
- Papers：

## 复审条件
- 将来出现什么信号时要重新审视？
```

## Session Handoff 模板

```markdown
# YYYY-MM-DD: 主题

## 改了什么
- 更新了哪些文件
- 新增了哪些 Ideas
- 跑了哪些 Experiments

## 学到了什么
- 发现 1
- 发现 2

## 未解决问题
- 问题 1
- 问题 2

## 下次优先阅读
- project_context.md
- 相关的 idea / experiment / decision 文件

## 推荐下一步
- 下一次会话开头的第一条具体动作
```
