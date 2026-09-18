# CUA（trycua/cua）项目评估

项目：<https://github.com/trycua/cua>

评估时间：2026-09-18。项目处于多组件独立版本、Nightly 高频发布阶段；仓库约 22.9k stars、1.6k forks，MIT License。

## 结论

CUA 是一套 Computer-Use Agent 基础设施，不是单一模型或浏览器自动化库。它解决的核心问题是：让不同多模态模型通过统一 API 操作一个真实桌面，并把桌面放在本地 VM、Linux 容器或云端隔离环境中，而不是直接放任模型控制用户宿主机。

它的主要价值是把 Computer Use 拆成可替换的四层：模型/Agent Loop、统一 Computer API、桌面执行 Server/Driver、隔离或托管的 Computer Provider。相较只做浏览器的 Browser Use/Playwright，它能操作完整 OS 和原生应用；相较直接使用 Anthropic/OpenAI Computer Use，它提供模型与运行环境解耦；相较 E2B，它更聚焦 GUI 桌面而非纯代码沙箱。

项目能力强、覆盖面广，但仍在快速演进。公开 Issue 约 989 个，近期大量问题集中在 macOS Accessibility、Windows UIA、Linux/X11 后台输入、VM 生命周期、动作成功误报和 benchmark 环境可复现性。适合隔离 spike 和非关键自动化，不应直接控制主力 Mac、已登录高敏账号或不可逆业务流程。

## 解决的问题

Computer-Use Agent 要真正完成“打开应用、点击、输入、读取结果”，至少要解决：

1. 不同模型的动作格式不一致；
2. 浏览器截图、键鼠、窗口、文件、Shell 和 OS 生命周期缺少统一接口；
3. 本地 macOS/Windows/Linux、Docker 和云桌面的启动方式不同；
4. 直接控制宿主机风险太高，任务之间也会污染状态；
5. Agent Loop 需要截图/可访问性树反馈、动作执行、重试、轨迹和评测；
6. 需要在开发时本地运行，扩容时切云端，而不重写 Agent。

CUA 试图提供类似“Computer Use 的基础云与设备抽象层”。

## 核心架构

### 1. Agent / Model 层

Agent 接收任务，调用支持视觉与工具使用的模型。模型观察屏幕或结构化桌面状态，输出点击、输入、快捷键、滚动、等待、Shell、文件等动作；Agent Loop 执行动作，再把新的观察送回模型，直到完成或触发停止条件。

CUA 的价值不是训练自己的通用模型，而是适配多个模型/Provider，并允许比较不同模型在同一 Computer 环境中的表现。模型能力、Computer 环境和执行 Driver 可以独立替换。

### 2. Computer API

统一暴露生命周期与交互能力，例如：

- 启动/停止/暂停/恢复 Computer；
- 获取屏幕、截图和显示尺寸；
- 鼠标移动、点击、拖拽、滚动；
- 键盘输入和快捷键；
- 文件上传下载、Shell/命令；
- 窗口和原生 UI 元素状态；
- 状态快照、网络/端口等 Provider 能力。

Agent 不需要知道背后是 Apple Silicon VM、Docker 还是云端实例。

### 3. Computer Server / Driver

桌面内部或宿主侧的 Server/Driver 把统一动作映射到具体 OS：

- 截图/视频流；
- 键鼠注入；
- macOS Accessibility（AX）；
- Windows UI Automation（UIA）；
- Linux X11/桌面辅助功能；
- 窗口枚举与后台输入；
- 文件和命令通道。

传统 Computer Use 主要靠像素坐标。CUA 新的 Driver 路线同时尝试提供可访问性树与元素级动作，从而减少截图 token、提高定位稳定性，并支持不抢用户前台焦点的后台操作。但这正是当前 Bug 最密集的区域。

### 4. Computer Provider / Sandbox

CUA 将“在哪里运行桌面”抽象为 Provider：

- macOS：Lume 在 Apple Silicon 上管理 macOS/Linux VM；
- Linux：Docker/容器桌面；
- 云端：Cua Cloud 提供托管的 macOS/Linux/Windows 沙箱；
- 其他本地或远端 Provider 可接入同一接口。

因此开发者可先在本地 VM 测试，再切云端并行扩容。

## 完整执行链路

```text
用户任务
→ Agent 选择模型并创建 Computer
→ Provider 启动 VM/容器/云桌面
→ Computer Server/Driver 建立控制通道
→ 截图或读取 Accessibility/UIA 状态
→ 模型判断下一动作
→ Computer API 执行点击/输入/窗口/文件操作
→ 获取新观察和动作结果
→ 循环直到目标、失败或预算上限
→ 保存轨迹/截图/结果
→ 销毁或重置沙箱
```

真正可靠的停止条件不能是模型说“完成了”，而应检查目标应用或后端状态。

## 主要组件价值

- **Computer SDK**：统一桌面生命周期与 I/O，隐藏 Provider 差异。
- **Agent SDK**：模型适配、Computer-Use Loop、消息与轨迹。
- **Computer Server**：运行在 Computer 环境内，提供截图、键鼠、Shell、文件等控制面。
- **Cua Driver**：更接近原生 OS 的窗口/可访问性/后台输入抽象。
- **Lume**：Apple Silicon 上以 Virtualization.framework 管理轻量 macOS/Linux VM，是本地 macOS 隔离的关键。
- **Cua Cloud/Sandbox**：托管桌面、并行伸缩和远程访问。
- **MCP/CLI/Examples**：让 Claude Code、其他 Agent 或自定义应用复用 Computer 工具。
- **CUA Bench**：在固定桌面任务上比较模型、Agent 和环境，但公开 Issue 显示部分环境镜像、网络绑定和 Oracle 仍有可复现性问题。

## 优势

### 1. 完整桌面，不局限浏览器

能覆盖 Finder、系统设置、Office、Electron、原生 macOS/Windows 应用，以及浏览器。Playwright 只能稳定操作浏览器 DOM；遇到桌面应用、系统对话框或跨应用流程时无能为力。

### 2. 模型与环境解耦

可以在同一 Computer API 上替换 Anthropic、OpenAI、Gemini 或开源视觉模型，也可以在不改 Agent 逻辑时把本地 VM 换成云沙箱。避免把业务深度绑定某一家 Computer-Use API。

### 3. 隔离优先

用 VM/容器/云桌面承载 GUI 自动化，理论上比直接控制个人桌面安全、可重置、可并行。每个任务可以有独立磁盘和登录态，结束后销毁。

### 4. macOS 本地 VM 是差异化能力

Lume 基于 Apple Virtualization.framework，在 Apple Silicon 上提供 macOS VM 管理。很多沙箱平台只做 Linux；如果任务必须验证 Safari、Xcode 或 macOS 原生 UI，这一能力有稀缺性。

### 5. 像素 + 原生可访问性双路线

纯截图 Agent 每一步都要发送图像，成本高且坐标脆弱。Driver 尝试读取 AX/UIA 树并按元素操作，可降低视觉 token，并提高按钮、菜单和文本框的可解释性；必要时再退回截图。

### 6. 本地到云端的一致开发面

本地调试、CI 和云端批量执行共享 SDK，有利于先小规模验证，再扩展并发，而不是重写一套云自动化。

## 与其他方案比较

### Browser Use / Playwright

- Browser Use：以浏览器 Agent 为中心，利用 DOM 与视觉；简单网页任务更轻、更成熟。
- Playwright：确定性浏览器自动化，适合已知流程和测试，速度、成本、稳定性通常最好。
- CUA：覆盖完整 OS 与跨应用任务，但更慢、更贵、状态空间更大。

原则：能用 API 就不用 GUI；能用 Playwright 就不用桌面像素 Agent；只有跨应用或无 API/DOM 时才用 CUA。

### Anthropic/OpenAI Computer Use

官方 Computer Use 提供模型动作协议，但不解决完整环境供应、VM 生命周期、跨 Provider 抽象和企业隔离。CUA 把这些补齐，并允许替换模型；代价是多一层 SDK 和兼容风险。

### OpenHands

OpenHands 主要面向软件工程，核心环境是仓库、Shell、编辑器与浏览器。CUA 更像通用桌面执行基础设施；做编码任务优先 OMP/OpenHands 类工具，做真实 GUI/跨应用验证才使用 CUA。

### E2B

E2B 擅长云端代码沙箱和进程执行。CUA 擅长带完整桌面、屏幕和原生应用的 Computer。两者可互补：代码步骤走 E2B/容器，GUI 步骤走 CUA。

### 传统 RPA

RPA 流程预先定义、确定性高，适合稳定重复流程；CUA 用模型动态理解界面，适合页面变化、自然语言任务和长尾流程。高频稳定流程最终应从 Agent 探索迁移为 API/RPA 脚本，而不是永久支付视觉推理成本。

## 成熟度判断

正面信号：

- 约 22.9k stars、1.6k forks，MIT License；
- 2025-01 创建，开发与 Nightly 发布非常活跃；
- Python、TypeScript、Swift/Rust/原生组件并存，覆盖 SDK、Server、Driver、Lume、云沙箱和 benchmark；
- 有多平台 CI、单元/集成/E2E、版本发布和安全相关检查；
- 实际问题反馈密集，说明已有较广泛试用。

风险信号：

- 多组件独立版本，Driver 当前仍在 0.28.x Nightly、Sandbox 等另有版本，整体 API 稳定性难用一个版本表达；
- 约 989 个 Open Issues；
- 近期核心问题包括：macOS AX 动作误报、元素树不完整、菜单/输入失败、每次动作额外等待约1秒；Windows UIA 超时和快捷键失败；Linux/X11 输入丢失、重复输入甚至桌面冻结；Lume 长时间运行/关闭崩溃；VM 启动错误被 202 掩盖；SSH 参数转义；VNC 绑定地址；benchmark 默认镜像不可公开获取等。

它不是“装上就能稳定代替人工桌面操作”的成熟 RPA 产品。

## 安全与成本风险

1. **屏幕泄露**：截图可能包含邮箱、聊天、Token、客户数据和通知。
2. **登录态风险**：沙箱浏览器/桌面中的 Cookie、Keychain 和 OAuth Session 可被任务使用。
3. **Prompt Injection**：网页、邮件、文档和应用文本可诱导 Agent 执行越权操作。
4. **宿主机风险**：本机 Driver 或宽目录挂载削弱 VM 隔离；虚拟化逃逸虽然概率低但影响高。
5. **网络外传**：Agent 可通过浏览器、上传表单、剪贴板或命令通道泄露数据。
6. **动作误报**：Driver 可能报告成功但应用没有接受输入，也可能重复执行。
7. **不可逆动作**：发送消息、购买、删除、发布和权限修改必须有人确认。
8. **成本/延迟**：连续截图、视觉模型、多轮等待和 VM 启动都昂贵；Accessibility tree 能优化但尚不稳定。
9. **供应链**：VM 镜像、远程桌面 Server、原生 Driver 和模型 Provider 都进入信任边界。

## 对 OpenClaw + OMP 的适配

- OpenClaw 适合做触发、审批、状态、会话路由和最终通知；
- OMP 继续负责代码实现；
- CUA 仅作为需要真实桌面验证的执行器，例如启动构建后的应用、操作 GUI、截图和验证结果；
- 不应让 CUA 替代 Playwright/API，也不应把它直接装成默认高权限工具。

建议采用明确的能力阶梯：

```text
API/CLI → Playwright/DOM → Accessibility 元素动作 → 截图坐标 Computer Use → 人工接管
```

每向右一级，成本和风险都增加。

## 最小隔离 Spike

1. 固定一个稳定 tag/commit，不跟 Nightly；使用专用测试 Mac 或本地 macOS VM，绝不控制主力桌面。
2. 创建低权限测试账号；VM 内不登录私人 Apple ID、邮箱、GitHub、支付或公司 SSO。
3. 默认禁外网，只允许测试站点；不挂载 Home、SSH、Keychain 和真实项目目录。
4. 只给 Agent screenshot/window/element/click/type 等最小工具，禁 Shell、文件上传和剪贴板出站。
5. 测三个任务：浏览器表单、Electron 设置项、跨两个测试应用复制非敏感文本。
6. 每个任务分别用 Playwright、CUA Accessibility、CUA Vision 跑 20 次，记录成功率、P50/P95、模型 token、动作数、误动作和人工接管。
7. 验证结果读取后端状态或应用持久化状态，不相信模型自报成功。
8. 加 Prompt Injection 页面、弹窗、窗口切换、输入延迟和重复提交测试。
9. 所有发送、删除、购买、发布类动作要求 OpenClaw 人工审批；每任务设置最大步骤、时间和费用。
10. 达不到 ≥95% 成功率或明显优于 Playwright/RPA 时，不进入生产。

Verdict：**CUA 的抽象和本地 macOS VM 能力有明显优势，适合作为“最后一公里 GUI 验证/长尾自动化”平台；当前原生 Driver 和跨平台稳定性仍不足，不适合成为默认执行层。**
