# OMP-WeChat 架构原理

项目：<https://github.com/mentalfl0w/omp-wechat>

## 定位

OMP-WeChat 是运行在 Oh My Pi / Pi 进程内的 extension，把腾讯 iLink Bot API 作为微信传输层，把 OMP/Pi SDK 作为 Agent 引擎。它不是把微信消息转发给一个外部 CLI 子进程，而是在同一进程中为每个微信会话创建 SDK `AgentSession`。

## 数据流

```text
微信用户
  → Tencent iLink getupdates 长轮询
  → 访问控制 + 跨进程消息去重
  → 每 chat 的 AgentSession.prompt()
  → OMP/Pi 模型和工具执行
  → message_end 提取文本
  → 去 Markdown、按 2000 字分段
  → iLink sendmessage 回复
```

图片和文件通过微信 CDN 下载并 AES 解密。支持视觉的模型接收 base64 图片；文本类文件提取内容进入 prompt。AI 写入每 chat 独立 outbox 的新文件会被 AES 加密上传 CDN，再通过 iLink 发送。

## 会话与进程

- 每个微信 chat 对应独立、可持久恢复的 OMP/Pi session。
- SessionPool 用 LRU 限制内存，默认最多 50 个活跃会话。
- 多个 OMP/Pi 进程可能同时加载 extension，因此用本机 TCP 端口 19821 作为 singleton lock；抢到端口者执行轮询，其他进程每 30 秒尝试接管。
- 开机服务运行 `omp --mode rpc`，每 5 秒向 stdin 写 `get_state` JSON-RPC 心跳，防止 RPC 模式因 stdin 空闲退出；systemd/launchd 再负责崩溃重启。

## iLink 约束

- 回复依赖最近入站消息的 `context_token`，不能主动发起会话。
- 仅支持 1:1，不支持微信群。
- 一个账号只允许一个 Bot 连接。

## 安全边界

- 默认 `pairing`，陌生人需要在 OMP 终端批准；也支持 allowlist 和 disabled。
- 微信凭据保存到 `~/.omp-wechat/credentials.json`，模式 0600；模型凭据复用 OMP/Pi auth，不由插件读取管理。
- 出站文件只能来自每 chat 独立 outbox，避免直接发送任意系统路径。
- 目前细粒度的“每用户工具权限”和“微信审批 bash”仍在路线图中。即使启用 pairing，被授权用户本质上仍可驱动 Coding Agent；应设置受限 `cwd`、最小工具权限，并视为高权限入口。

## 当前部署注意

项目生成的 Linux systemd 服务只设置 `/usr/local/bin:/usr/bin:/bin`。若 `omp` 安装在 `~/.bun/bin` 或 `~/.local/bin`，服务可能找不到命令，需要在安装前修正为绝对路径或调整服务 PATH。项目 README 声称 OMP 是自包含二进制，但具体安装方式可能仍依赖 Bun，应以目标机器的 `file`、shebang 和 `command -v omp` 实测。
