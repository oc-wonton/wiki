# 阅读：Pi 与 Claude Code 的两条 Agent Harness 路线

原文：[我把 pi 的源码读完了：一条和 Claude Code 完全相反的路线](https://mp.weixin.qq.com/s/Ye4VwMR3XGldD-qGWTjMGw)，作者 lebhoryi / 一只在路上的哈士奇。

> 注：微信正文受反爬限制，本次只能取得标题、搜索摘要，并以 Pi 官方仓库、system prompt 与 agent loop 源码交叉验证核心论点；不是逐段全文摘录。

## 核心论点

Pi 的原始路线是极简、透明、可组合的 Agent Harness：默认只给模型 read、bash、edit、write 四个工具，不内置权限系统、强制计划模式或复杂编排。Agent 的“下一步做什么、何时停止”主要由模型通过普通 tool-calling 决定；核心循环只是发送上下文、执行工具调用、把结果放回上下文，直到模型不再调用工具。

Claude Code 代表更产品化的路线：围绕模型增加权限、计划、任务管理、子代理、hooks、IDE 集成和大量策略。前者强调“最小内核 + 用户扩展”，后者强调“完整工作台 + 默认护栏”。

## 源码交叉验证

- Pi 默认工具确为 `read / bash / edit / write`。
- system prompt 很短，核心只是说明工具、简洁回复、清晰展示路径，再叠加项目 context 与 skills。
- agent loop 是标准闭环：模型响应 → 检查 tool calls → 执行 → 追加 tool results → 再调用模型；没有隐藏 planner 决定每一步。
- Pi 官方明确说明没有内置文件、进程、网络或凭据权限系统，默认继承启动用户权限；需要隔离时应使用微型 VM、Docker 或 policy sandbox。

## 判断

“完全相反”适合作为产品哲学对比，但不应理解为二者在技术结构上毫无共同点。两者底层都是上下文、模型、工具和循环；差异在于多少控制逻辑放进 harness 默认层。

Pi 的优势是可读、可改、模型无关、适合学习和嵌入；代价是默认安全边界弱，生产使用必须外置 sandbox 和治理。Claude Code 的优势是开箱即用、护栏和工程体验完整；代价是系统更复杂、行为更不透明、定制时更容易受框架约束。

## 对当前选择的启示

OMP 不是“原味 Pi 的极简主义”本身，而是在 Pi 底座上重新加入 LSP、DAP、多 Agent、权限、记忆和 IDE 能力。因此选择 OMP 的合理理解是：保留 Pi 的可扩展内核，同时采用更完整的工程工具面。不要因为 Pi 极简就把所有功能都塞回核心；通过可替换 skill、extension 和外部 sandbox 分层更合适。
