# Laya 项目评估

项目：<https://github.com/NandhaKishorM/laya>

评估时间：2026-09-21；源码快照：`42626c348753fbb17572a813127df2278a1ec527`；版本 `0.3.4`。

## 定位

Laya 是开源、自托管的非自回归 System 1 决策模型，定位接近 TypeSafe AI 的 Jev：输入一段状态和预定义的 typed questions，一次前向传播并行输出 choice、ordinal score 或二元概率，以及各选项概率和置信度。它不生成自然语言，不是聊天模型、Coding Agent 或 RAG。

它解决的是软件内部高频“模糊 if”问题：分类、路由、风险评分、guardrail、内容审核、工单分派和 Agent 动作建议。普通 LLM 为这些小决策逐 token 生成 JSON，延迟和成本过高，格式与置信度也不稳定；Laya 用 Encoder + Decision Heads 直接给类型化概率。

## 执行方式

```text
state + questions schema
→ tokenizer 将状态、问题、候选项拼成一个序列
→ ModernBERT/mmBERT Encoder
→ 每个候选 marker 的并行 logits
→ temperature calibration
→ choice/score/noul + probabilities + confidence
```

一个调用中的多道问题被组成 batch，在一次模型 forward 中完成。输出 token 为 0；成本主要是输入编码与一次推理。

## 三种 primitive

- `choice`：有限候选项中选择，返回 Top label 和完整概率分布；
- `score`：有序等级，返回期望分数和分布；
- `noul`：二元 yes/no 概率。

内置 presets 覆盖客服 triage、邮件、prompt guard、moderation 和模型路由。

## Checkpoint 与 Router

- English：ModernBERT-large，421M，512 context；
- Multilingual：mmBERT-base，322M，1024 context；
- Typed decisions：ModernBERT-large，421M，针对四个公开 workflow fine-tune。

Router 先用脚本/语言检测在毫秒以下选择 English 或 Multilingual；Typed checkpoint 默认不自动选择，除非显式 task/model 或打开 workflow 签名检测。模型冷加载需数秒，语言流量切换时应 preload；三个模型共约 1.16B 参数，会占明显 RAM/VRAM。

## 真实优势

- Apache-2.0，权重公开，可本地运行；
- 单 T4 一问约 32.8–39.5ms，10 问约 72–159ms；
- 多问题单 forward，远快于逐 token LLM；
- 输出 Schema 固定，不会出现解析失败或枚举外字符串；
- 可按业务数据 fine-tune，并做 temperature calibration；
- 安全上使用 safetensors，CI 禁止 pickle/torch.load、shell-out 和动态 exec。

## 关键边界

1. “没有 hallucination”只成立于格式/枚举层面，语义判断仍会错。
2. 基础 checkpoint 在 typed-decisions zero-shot 接近随机且低于多数类基线；0.766 是在该 benchmark 训练 split 上 fine-tune 后的结果。它更像可专门化底座，不是通用零样本决策引擎。
3. English 模型在非英语上会高置信崩溃，必须依赖外部 Router；模型自身 confidence 无法救场。
4. 两个基础 checkpoint 原始概率过度自信；README 的低 ECE 依赖按问题类型/选项数重新拟合 temperature，多语言模型默认没有 fitted temperatures。
5. 高基数候选（50+）受 head token budget 限制，Banking77 仅 0.425，而 Jev 报告 0.870；需要扩大 head_max_len 或分层选择。
6. Ordinal score 是弱项；长文本 Context 仅 512/1024 tokens，复杂推理和开放输出不适用。
7. Benchmark 主要由作者自建；与 Jev 的数据来自第三方、Prompt/样本规模不同，不是严格同场对比。

## 成熟度

仓库创建于 2026-09-18，三天内约 5.3k stars、474 forks，属于刚爆红的极早期项目。只有 7 次 release，最新 0.3.4；核心包仅约 7 个 Python 模块，测试主要是 Router、criteria 和本地 E2E。已有 CI、Docker、Release、Security、CodeQL 和依赖审计，但部署 API、Docker、GGUF、TypeScript、MCP、高基数与 calibration 功能仍在 PR/Issue 中。

## 与 Jev 的关系

Laya 是开源可自托管路线，理念和 API 受 TypeSafe Jev 影响。Jev 是闭源服务，宣称原生 RLCD 架构、70–500ms、最多255候选；Laya基于公开 Encoder 架构和 fine-tuning，低基数任务速度可能更快、成本可控，但零样本能力和高基数表现弱，且需要自行校准和部署。

## 对当前体系的适配

适合在 OpenClaw/OMP 流程中作为廉价前置分类器或 policy head：

- PR/Issue 风险分类与模型路由；
- 日报候选粗筛；
- 大规模 tool trace 的 needs-review / outcome / risk 分类；
- Computer Use 中有限动作集合建议；
- Prompt guard 初筛。

不应直接决定发送、删除、合并、支付或权限变更；低置信必须升级到 `gpt-5.6-sol` 或人工，且最终由 CI/代码规则验收。

## 建议 Spike

选一个有至少数千条历史标签的窄任务，分 train/calibration/test 三组，比较 Laya、规则基线和 `gpt-5.6-sol-fast`：Accuracy/F1、Brier、ECE、覆盖率-错误率曲线、P50/P95、RAM/VRAM和每千次成本。必须单独拟合 temperature，并使用未参与 fine-tune 的 holdout；无法达到业务容错要求就只做辅助信号。

Verdict：**思路正确、速度有吸引力，适合窄域高频决策实验；目前太新，基础模型零样本能力弱，不能把 README 的 typed-decisions 结果当成通用能力。**
