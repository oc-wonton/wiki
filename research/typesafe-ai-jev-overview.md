# TypeSafe AI / Jev 概览

来源：<https://typesafe.ai/>、<https://typesafe.ai/blog/introducing-system-one-models-and-jev>

评估时间：2026-09-18。

## 定位

TypeSafe AI 是一家构建“machine-native intelligence”的 AI Lab。其首个公开模型 Jev 被定义为 System One Model：不生成自由文本，而是接收非结构化或结构化状态与预定义问题，直接并行输出类型安全的选择、概率与置信度，供软件流程自动分支。

它瞄准的不是聊天、写作、代码生成和开放式推理，而是高频、低延迟、可组合的 fuzzy decision function：分类、路由、打分、抽取、guardrail、judge、异常检测和实时策略判断。

## 与 LLM structured output 的差别

普通 LLM 仍逐 token 生成字符串，再由 JSON mode/schema constraint 保证外壳可解析；内容本身可能过度自信、不一致。Jev 声称模型架构和采样原生围绕有限类型输出，所有结果并行生成，并返回校准概率。

“零 hallucination”只应理解为 schema/type 层面不会生成枚举外值，并不代表语义决策永远正确。官方也承认 Jev 会出错，价值在于置信度校准后允许软件按阈值自动执行或升级人工。

## 官方主张

- RLCD（Reinforcement Learning for Calibrated Decisions）训练；
- 70–500ms 端到端延迟；
- 输入 $0.042/百万 token，输出不计费；
- 官网工作流示例声称 193.6× 更快、444.6× 更便宜；
- 高基数输出最多 255 个选择，超出时使用两阶段评分/选择。

这些数字来自 TypeSafe 自建 workflow eval，参考答案取 GPT-6 Astra 与 Claude Fable 5.1 平均，并用其 System One adapter 约束 LLM 输出。官方主动披露短输入、工作流作者偏差、比较方式可能放大优势等限制。因此应视为厂商早期证据，而非独立 benchmark。

## 适用边界

适合：已预先定义输出空间、有大量并行判断、低延迟/低成本重要、且可以根据置信度决定自动执行或人工升级的工作流。

不适合：需要生成文本/代码、开放式规划、复杂多步推理、事实探索、工具操作或输出空间无法预先定义的任务。

## 与 Agent/Loop 的关系

Jev 更像 Loop 中的快速“政策头”或 evaluator，而不是 Agent 本身：

```text
观察状态 → Jev 输出 typed action probabilities → 代码阈值/策略 → 执行动作或交给强模型/人
```

在 Computer Use 中可用于从有限动作集合中快速建议下一步；在 PR Flow 中可用于风险分类、是否升级深度 Review；在日报中可用于候选分类。但最终动作仍需代码权限、确定性校验和必要人工审批。

## 建议

目前处于 early access。若获得权限，先选一个有历史标签的窄任务，与 `gpt-5.6-sol-fast`/规则基线比较准确率、ECE/Brier calibration、覆盖率、P95、每千次成本和人工升级率。不要用单一准确率，也不要把厂商“零 hallucination”解释为零业务错误。
