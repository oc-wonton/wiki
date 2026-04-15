# OpenCLI — 把任意网站/桌面应用变成 CLI

## 是什么
开源的统一 CLI Hub，把网站、Electron 桌面应用、本地工具都抽象成标准命令行界面。给人和 AI Agent 都能用。

- **Repo**: https://github.com/jackwener/opencli
- **官网**: https://opencli.info
- **作者**: jackwener（也是 wechat-article-to-markdown 作者）
- **协议**: Apache-2.0
- **Stars**: 7.3k+

## 核心架构

```
opencli <site> <command>  ← 统一入口
        │
  ┌─────┼──────────┐
  ▼     ▼          ▼
Website  Desktop   Local CLI
Adapter  Adapter   Hub
(87+)   (8 apps)  (gh, docker...)
  │      │          │
  ▼      ▼          passthrough
  Chrome DevTools Protocol (CDP)
        │
  Browser Bridge Extension (Chrome 扩展)
        │ HTTP :19825
  Local Daemon (本地守护进程)
```

### 三种数据源
1. **Website Adapters** — JS adapter 注入页面 evaluate()，复用浏览器登录态
2. **Desktop App Adapters** — CDP 控制 Electron 应用（Cursor/Codex/ChatGPT/Notion/豆包等）
3. **Local CLI Hub** — passthrough 本地二进制，统一发现 + 自动安装

## Adapter 生成流程
```
explore → synthesize → generate → register
```
- explore: 打开页面，抓网络请求，分析 API
- synthesize: 探索结果 → JS adapter 代码
- generate: 端到端一键生成
- cascade: 认证探测（PUBLIC → COOKIE → HEADER）

## 关键设计
- **确定性 > 智能**: 运行时零 LLM 成本，AI 只在生成 adapter 时参与
- **复用浏览器登录态**: credentials never leave the browser
- **Unix 哲学**: sysexits.h exit code，输出可 pipe/grep/jq

## 内置 87+ 适配器
B站、小红书、知乎、Twitter、Reddit、HackerNews、1688、闲鱼、Spotify、微信文章等

## 与 Webify-MCP 对比
| 维度 | OpenCLI | Webify-MCP |
|---|---|---|
| 方式 | 主动控制（CDP 注入脚本） | 被动拦截（webRequest） |
| 数据 | evaluate + DOM | 后端 API 原始 JSON |
| 输出 | CLI（人+Agent） | MCP（Agent） |
| 适配器生成 | AI 辅助（explore→synthesize） | 手写 JSON Schema |
| 浏览器控制 | ✅ 全套操作 | ❌ 纯拦截 |

互补关系：Webify 数据质量更高，OpenCLI 控制能力更强。

## 收录时间
2026-04-14
