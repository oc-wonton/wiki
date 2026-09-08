# 阅读：Loop engineering — Getting started with loops

原文：<https://claude.com/blog/getting-started-with-loops>

作者：Delba de Oliveira、Michael Segner（Claude Code 团队）；发布日期：2026-06-30。

## 定义

Claude Code 团队把 Loop 定义为：**Agent 重复执行工作周期，直到满足停止条件。** 分类维度包括触发方式、停止方式、使用的 Agent primitive，以及适合的任务类型。

Loop Engineering 不是默认把任务做复杂。文章反复强调：先使用最简单方案，只在结果确实改善时增加循环、多 Agent 或主动调度。

## 四种 Loop

### 1. Turn-based loop

- 触发：用户实时发出 Prompt。
- 停止：Agent 判断任务完成，或需要更多上下文。
- 适合：短任务、一次性任务、非固定流程。
- 管理方式：明确 Prompt，并用 Skill 加强自验证，减少人工来回。

普通 Coding Agent 对话已经是 Loop：读取代码、修改、运行测试、根据结果继续修复，最后交付。文章建议把用户每次手工检查的步骤编码成 Skill，并给 Agent 能观察、测量和操作真实结果的工具。验证越定量，Agent 越容易自检。

### 2. Goal-based loop

- 触发：用户实时给出任务和目标。
- 停止：目标达成，或达到最大轮数。
- 适合：存在可验证退出条件的复杂任务。
- 管理方式：明确“完成”的定义，并设置 turn cap，例如最多尝试五次。

与普通对话的关键区别是，不让 Agent 自己决定“够好了”。每次 Agent 想停止时，由 evaluator 检查目标；不满足就返回继续工作。测试通过数、分数阈值等确定性条件尤其适合。

### 3. Time-based loop

- 触发：固定时间或间隔。
- 停止：人工取消，或外部工作完成，例如 PR 合并、队列清空。
- 适合：重复工作，或轮询外部系统变化。
- 管理方式：降低不必要的检查频率；能事件驱动就不用高频轮询。

同一任务、不同输入的日报属于定时 Loop；等待 PR Review、CI 状态则属于轮询外部系统。Claude Code 本机 `/loop` 会随电脑关机停止，云端 `/schedule` 用于持久调度。

### 4. Proactive loop

- 触发：事件或计划，无实时人工参与。
- 停止：单个任务达到目标；Routine 本身持续到被关闭。
- 适合：持续到来的、边界清晰的工作流，例如 Bug 报告、Issue Triage、迁移和依赖升级。
- 管理方式：常规步骤使用更快、更便宜的模型，把强模型留给判断节点。

它通常组合 schedule、goal、skills、dynamic workflows 和自动权限模式：定时找新输入，以可验证目标定义完成，多个 Agent 分工处理和复核，然后自动执行低风险操作。

## 质量保障

文章认为 Loop 的输出质量主要取决于周边系统，而不是重复次数：

1. 代码库本身保持干净，因为 Agent 会模仿现有模式。
2. 用 Skill 编码“什么叫做好”，并给 Agent 可验证真实结果的工具。
3. 文档必须容易获取且保持最新。
4. 写代码的 Loop 需要独立检查代码的 Loop；Reviewer 应使用新上下文，避免被实现 Agent 的推理锚定。
5. 单次结果不好时，不只修该结果，还要把纠正沉淀进未来所有迭代的检查或流程。

## 成本与边界

- 为任务选择最简单的 primitive 和足够的模型，不默认多 Agent。
- 明确成功条件与停止条件，防止过早结束和无限迭代。
- 大批量运行前先用小样本 Pilot；动态 Workflow 可能启动数百 Agent。
- 确定性工作写成脚本，比每次让模型重新推理更便宜、更可靠。
- 调度频率应匹配数据变化频率。
- 持续观察 Skill、Subagent、工具调用、轮数和 Token 使用，并能随时停止。

## 判断

文章最重要的贡献是给“Loop”去神秘化：普通 Agent 对话本身就是 Loop；高级 Loop 只是改变触发器和停止条件。真正的设计中心不是 Prompt，而是：

```text
触发器 + 可验证目标 + 反馈信号 + 停止条件 + 预算与权限边界
```

局限是文章主要描述 Claude Code primitives，并未深入处理并发冲突、补偿事务、Exactly-once、错误分类、失败告警、人工审批安全和生产状态恢复。这些需要由 OpenClaw、CI、队列和业务系统补齐。

## 对当前体系的映射

- 日常对话和 OMP 编码：Turn-based Loop。
- OMP 修复 Bug，最多三轮且测试必须通过：Goal-based Loop。
- 北京时间 09:00 AI 日报：Time-based Loop。
- 自动接收 Issue、分类、实现、Review 和送审：Proactive Loop。

AI 日报的完整停止条件不能只是“内容已生成”，而应是：日报文件生成、去重账本更新、Git push 成功、微信送达状态为 delivered；任一步失败都进入补偿或告警。过去出现的 `prepare failed` 和 `NO_REPLY` 正说明触发器存在不等于闭环成立。

OMP 编码 Loop 建议使用统一合同：

```yaml
goal: 明确的最终结果
verify: 测试、lint、构建和验收命令
max_turns: 3
max_duration: 30m
max_diff: 按任务定义
permissions: 最小工具和目录范围
stop_on: 目标达成、预算耗尽、风险边界、不可恢复错误
escalate_to_human: 需求变化、认证/数据迁移、安全冲突、连续两轮无进展
```

文章与 QQ 飞车实践的共同点是“概率生成、确定性验收”；后者进一步补足了反馈持久化和无界 Loop 的真实失败案例。
