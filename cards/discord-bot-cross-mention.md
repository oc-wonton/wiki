# Discord Bot 互相 @ 通信

## 问题
多个 OpenClaw bot 在同一个 Discord server 无法互相 @ 触发

## 解决方案

### 1. 去掉 users 白名单
不要在 discord channel 配置里写 `"users": ["xxx"]`，否则其他 bot 的消息会被过滤。

### 2. 加 allowBots
```json
"discord": {
  "allowBots": "mentions"
}
```

### 3. @ 格式必须正确
- ✅ `<@USER_ID>` — 真正的 user mention
- ❌ `<@&ROLE_ID>` — 这是 role mention，不触发 user mention
- ❌ `@name` 纯文本 — Discord 不解析

### 4. respondToBots 位置
只在 `channels.discord` 下有效，不要放在 acp、hooks、plugins 等其他位置，否则配置校验失败。

## Pitfalls
- SSH 远程改别人配置文件容易改错位置
- 配置校验失败时 gateway reload 会被静默跳过，日志里才能看到
