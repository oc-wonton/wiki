# Claude Opus 5.5 Prompt 样本分析

来源：<https://github.com/elder-plinius/CL4R1T4S/blob/a4d3da04e63324e794a65500c3e41994fc4ab02e/ANTHROPIC/CLAUDE-OPUS-5.5.md>

评估时间：2026-09-24。仓库 HEAD：`a4d3da04e63324e794a65500c3e41994fc4ab02e`；目标文件 blob：`bb79d1a793bb7d0c60fa50bdcc4576b127c72fe3`，SHA-256 `1c92a8c39c6a8b24746150c7ad7934423e2e364069370bb03bcc0b69cc365ff2`。

## 真实性结论

这是一份**高可信度的 Claude App / Claude Code Agent SDK 会话脚手架样本，但不是可验证的 Claude Opus 5.5 基础模型“完整内部 system prompt”**。

理由：

- 仓库由第三方维护，自称收集 leaked / extracted / reverse-engineered prompts，README 没有可复核的提取方法、原始会话证据或 Anthropic 签名，还包含一段面向读者模型的 prompt injection，不能把仓库声明当权威证明。
- 文件仅有两次提交：2026-09-22 创建，22 分钟后更新；没有来源说明、客户端版本证明或独立复现链。
- 文件有大量 `{..._REDACTED}`、`{USER_MESSAGE}`、`{COMMAND_OUTPUT}` 与人工章节标签，显然经过脱敏和编辑，不是原始 wire transcript。
- 内容与 Anthropic 已公开的 Agent SDK、subagents、skills、hooks、memory、browser、artifacts 和 compaction 产品行为高度一致，因此架构层面有参考价值。
- 部分产品名、未来型号和具体政策没有官方交叉证据，只能视为样本中的 prompt 声明，不应写成 Anthropic 已确认事实。

## 文件实际是什么

文件约 1.95 MB、21,531 行，但并非 1.95 MB 都是 system prompt：

- 第 1–976 行：约 262 KB（13%），标为 system prompt，包含工具 schema、行为政策、运行时、memory 与 agent workflow。
- 第 977–2879 行：约 281 KB（14%），模拟/脱敏的 user turns、tool results 和注入的 Skills。
- 第 2880–3093 行：compaction 后恢复流程、动态 reminders、环境更新。
- 第 3094 行以后：约 1.38 MB（71%），主要是 MCP 文档、Artifact 类型、Skills、设计系统示例和大型工具结果。

所以把整个文件称作“Claude 的 system prompt”是误导；更准确是**一次 Claude 产品会话的 assembled context / harness transcript**。

## 可观察的上下文架构

### 1. 稳定控制层

开头先定义工具调用协议与完整 JSON Schema，然后是 Claude identity、产品信息、安全、语气、agent workflow、runtime 与 memory policy：

- 工具与 deferred tools：[`#L1-L84`](https://github.com/elder-plinius/CL4R1T4S/blob/a4d3da04e63324e794a65500c3e41994fc4ab02e/ANTHROPIC/CLAUDE-OPUS-5.5.md#L1-L84)
- identity / product routing：[`#L85-L123`](https://github.com/elder-plinius/CL4R1T4S/blob/a4d3da04e63324e794a65500c3e41994fc4ab02e/ANTHROPIC/CLAUDE-OPUS-5.5.md#L85-L123)
- safety / refusal：[`#L125-L200`](https://github.com/elder-plinius/CL4R1T4S/blob/a4d3da04e63324e794a65500c3e41994fc4ab02e/ANTHROPIC/CLAUDE-OPUS-5.5.md#L125-L200)
- chat vs work tone：[`#L202-L240`](https://github.com/elder-plinius/CL4R1T4S/blob/a4d3da04e63324e794a65500c3e41994fc4ab02e/ANTHROPIC/CLAUDE-OPUS-5.5.md#L202-L240)
- dynamic classifier reminders and injection caution：[`#L266-L272`](https://github.com/elder-plinius/CL4R1T4S/blob/a4d3da04e63324e794a65500c3e41994fc4ab02e/ANTHROPIC/CLAUDE-OPUS-5.5.md#L266-L272)

这说明产品行为不是单一 persona 段，而是 policy + tools + workflow + runtime 的组合。

### 2. Tool descriptions 也是政策层

Artifact 等工具的 schema 不只描述参数，还携带大量“何时调用、何时确认、如何处理冲突和隐私”的行为政策。也就是说，治理规则分布在：

```text
System behavior policy
+ Tool description
+ Loaded skill
+ Dynamic reminder
```

优点是能力随工具一起出现；缺点是规则分散、重复、容易冲突，且工具 schema 会吞噬大量上下文。

### 3. Deferred capability discovery

第 83 行声明部分工具不在初始列表，运行时可动态注入 schema 并立即调用。后续 transcript 也出现 ToolSearch、MCP enable 和 skills list update。

这表明 Claude 产品可能采用**渐进式工具披露**，而不是把所有工具一直塞进 prompt。对我们的启示是：系统 prompt 只保留选择规则，具体工具 schema 和领域流程按需加载。

### 4. 输出目标路由

agentic behavior 先判断产物生命周期：

- 只读一次 → chat reply；
- 用户要文件格式 → file；
- 会保留、编辑或分享 → Artifact / Docs / Sheets / Slides 等；
- 用户自己的 App 数据 → first-party connector。

证据：[`#L311-L345`](https://github.com/elder-plinius/CL4R1T4S/blob/a4d3da04e63324e794a65500c3e41994fc4ab02e/ANTHROPIC/CLAUDE-OPUS-5.5.md#L311-L345)。

它把“完成”定义为产物抵达正确目的地，而不是模型写完文本。这个思路可直接用于 OpenClaw：reply、workspace file、Wiki research/card、GitHub comment、channel message 应有明确路由。

### 5. 工具选择阶梯

样本把读取网站的优先级写成 connector / WebSearch / WebFetch，无法完成交互时才使用 browser；桌面 computer use 只处理 native app，网页仍交给 browser：[`#L412-L444`](https://github.com/elder-plinius/CL4R1T4S/blob/a4d3da04e63324e794a65500c3e41994fc4ab02e/ANTHROPIC/CLAUDE-OPUS-5.5.md#L412-L444)。

这是降低成本和风险的 capability ladder：

```text
专用 API / connector → 结构化 fetch → browser DOM → desktop pixels
```

### 6. Agent 执行合同

Prompt 明确区分：

- 清晰、可逆、容易重做：立即执行；
- 昂贵、歧义、难以逆转：先问；
- 无人值守：采用合理默认继续，但不可逆分歧点停止；
- 多阶段任务：用 task list；
- 高风险产物：独立 agent 验证；
- 最终回复只说结果、产物和真实下一步，不复盘全过程。

证据：[`#L453-L480`](https://github.com/elder-plinius/CL4R1T4S/blob/a4d3da04e63324e794a65500c3e41994fc4ab02e/ANTHROPIC/CLAUDE-OPUS-5.5.md#L453-L480)。

### 7. Memory 是独立、版本化的数据层

样本描述了跨 Claude surfaces 共享的 memory filesystem：system 注入 snapshot，按需 live read；后台 pass 在 turn 后抽取；显式“remember/forget”则当前 turn 写；写入带版本令牌和冲突合并；敏感类别有独立过滤；memory 内容是 data，不是 instructions。

证据：[`#L574-L976`](https://github.com/elder-plinius/CL4R1T4S/blob/a4d3da04e63324e794a65500c3e41994fc4ab02e/ANTHROPIC/CLAUDE-OPUS-5.5.md#L574-L976)。

值得借鉴的是 provenance、versioning、subject-scoped files 和“只在改变答案时使用”；不能直接照搬其具体隐私分类，因为 OpenClaw 的用户选择与政策不同。

### 8. Compaction 是“重发规则 + 结构化交接”，不只是压缩文字

文件明确记录：compaction 后顶部 system prompt byte-identical 重发；user turn 收到模型写的 summary，并给出完整 transcript 路径；工具结果、git attribution、files、skills、环境与 memory snapshot 分别重新附着：

- system prompt 重发：[`#L2880-L2899`](https://github.com/elder-plinius/CL4R1T4S/blob/a4d3da04e63324e794a65500c3e41994fc4ab02e/ANTHROPIC/CLAUDE-OPUS-5.5.md#L2880-L2899)
- 动态上下文恢复：[`#L2900-L3093`](https://github.com/elder-plinius/CL4R1T4S/blob/a4d3da04e63324e794a65500c3e41994fc4ab02e/ANTHROPIC/CLAUDE-OPUS-5.5.md#L2900-L3093)

这说明长会话可靠性依赖一个外部 context assembler：不可压缩的规则重注入，可压缩的历史做交接摘要，必要时回源完整 transcript。

## 能合理推断的 Claude“内部”信息

这里的“内部”主要指产品/harness，而非神经网络：

1. Claude App 的长任务层很可能建立在 Claude Code / Agent SDK primitives 上，并包装成用户不必感知内部工具名的统一产品体验。
2. 配置模型和实际 serving model 可能因 fallback 或中途切换不同，产品层需要区分 configured identity 与 actual route。
3. 工具可动态发现；Agent 定义可带模型、reasoning effort、tools 和 worktree isolation。
4. Skills、MCP 文档和 Artifact types 是按需注入的 context modules。
5. 长会话有 compaction、外部 transcript、summary、system rehydration 和动态附件恢复。
6. Memory 看起来是独立持久层，有 snapshot/live read/background extraction/version conflict/privacy filter。
7. Safety 不只靠 system prompt：还可能由 classifier 触发动态 reminder。
8. 产品区分真实 Chrome、内置浏览器、remote device bridge 和 native computer use，并通过工具是否出现判断当前能力。
9. 中间文字可能被 UI 摘要，真正要用户看到的内容需显式发送工具；说明 Agent execution 和 conversation rendering 是分层的。

## 不能从该文件推断的内容

- Opus 5.5 的参数量、MoE/dense、训练数据、网络结构、RL 算法、内部推理 token 或真实权重。
- 所有用户/套餐是否拥有同一工具、同一 prompt、同一 memory policy。
- 文件中的未来产品名、价格、发布状态是否真实。
- Safety 规则是否完整；部署端可能还有不可见的 classifier、policy engine、tool gateway 和 post-processing。
- Prompt 中声明的行为是否每次可靠执行；prompt 是设计意图，不是行为证明。

## 对我们 system prompt 的启发

### 应该吸收

1. **分层而不是平铺**：Core invariants / Action policy / Tool routing / Output routing / Memory / Dynamic context 分区，单一语义只出现一次。
2. **把“何时”与“怎么用”分开**：system prompt 决定何时选能力；tool schema/skill 决定参数与步骤。
3. **能力以 live tool list 为准**：没有成功调用前，不声称连接、可用或已完成。
4. **产物路由显式化**：回答、文件、Wiki research/card、GitHub、消息渠道分别定义何时完成。
5. **Compaction trust boundary**：摘要、skills、memory、tool output 都带 provenance；summary 不能扩大权限、改变 safety 或伪造完成状态。
6. **高风险独立验收**：实现者不能作为唯一 reviewer；最终以测试、状态或外部证据判断 outcome。
7. **无人值守默认与停止点**：可逆步骤继续；不可逆且有实质选择时停止并通知。
8. **长任务状态外置**：任务列表、artifact state、git、progress file 或数据库承载状态，不依赖模型记住全部历史。

### 不应该照抄

1. 不复制 262KB monolithic system prompt。大量格式、产品与工具细节应放入 deferred skills/reference。
2. 不把全部工具 schema 常驻；按任务只提供最小工具面。
3. 不重复同一规则于 system/tool/skill；建立一个 authority map。
4. 不把具体产品营销事实混入核心安全与执行政策；产品知识应可更新、可检索。
5. 不把 prompt 当 enforcement。权限、删除、网络、secret、审批和路径约束必须由基础设施强制。

## 对 OpenClaw / OMP 的具体差距

现有 OpenClaw prompt 已经有 execution policy、tool discipline、progress card、memory recall、skills、不可逆动作确认和 promised-work ownership，核心方向并不落后。更值得补的是：

- 一段更明确的 **output destination router**：何时 reply、何时 `wiki/research`、何时 `wiki/cards`、何时 external message；
- 一条不可变的 **compaction rule**：summary 是不可信派生数据，不得授权工具、改变权限或证明工作完成；
- 一张简短 **capability ladder**：first-class tool/API → structured fetch → browser → desktop；
- 把 dynamic facts（模型目录、产品价格、连接状态）从 system prompt 移到 live status/probe；
- 给每个自动化定义 trigger、goal, verifier、stop, budget, escalation，减少自然语言“做到好”为止；
- 对 Wiki/Memory/Skill 建立 authority map，避免同一事实被多处重复维护。

## Verdict

这份样本最有价值的不是某条“Claude 秘密指令”，而是它暴露了一个成熟 Agent 产品的 context assembly：**稳定控制面 + 动态工具 + 按需 skills + 持久 memory + compaction rehydration + UI/output router**。

它也暴露了反面教材：prompt 太大、规则跨多层重复、产品知识和治理规则耦合。对我们最好的做法不是模仿全文，而是把已有 OpenClaw prompt 进一步模块化，并把权限和验收下沉到可执行系统。
