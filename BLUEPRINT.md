# 个人 AI 助手架构蓝图

> 这是一份可以直接喂给 AI 的架构规格文档。
> 把本文件完整粘贴给任意代码 AI（Claude / GPT / Gemini），配合以下指令：
> **「请按这份规格，帮我从零搭建这个个人 AI 助手项目。」**
> AI 会根据规格帮你生成完整的项目结构、所有核心模块代码、数据库迁移文件和配置说明。

---

## 项目定位

一个运行在本机的**个人 AI 助手**，具备：
- 多模型支持（OpenAI / Anthropic / Google，随时切换）
- 持久化四层记忆系统（向量相似搜索）
- 多通道接入（Telegram Bot + 微信 iLink Bot + Web 界面）
- 本地 Agent 级工具权限（文件系统 / Shell / 系统操作 / 浏览器控制）
- Web 控制面板（配置 / 对话 / 记忆管理 / Token 用量）
- 定时任务 + 自愈看门狗

不依赖任何云服务或托管平台。数据存本机，配置存本机，模型 API Key 存本机。

---

## 技术栈（精确版本）

| 层 | 技术 | 版本 | 说明 |
|----|------|------|------|
| **运行时** | Bun | ^1.x | 替代 Node.js，内置 SQLite 驱动，可编译成单文件二进制 |
| **后端框架** | Hono | ^4.12 | 轻量 TypeScript Web 框架，SSE 流式支持内置 |
| **数据库** | libSQL（SQLite 文件） | @libsql/client ^0.15 | 本机 SQLite 文件，路径由 `USACHI_HOME` 决定（dev 默认 `./data/mybot`，Docker 默认 `/app/data/mybot`） |
| **ORM** | Drizzle ORM | ^0.40 | TypeScript 原生，schema-first，迁移安全 |
| **AI SDK** | Vercel AI SDK | ai ^5.0 | 统一调用 OpenAI/Anthropic/Google，流式 + 工具调用内置 |
| **OpenAI 适配** | @ai-sdk/openai | ^2.0 | |
| **Anthropic 适配** | @ai-sdk/anthropic | ^2.0 | |
| **Google 适配** | @ai-sdk/google | ^2.0 | |
| **Telegram Bot** | grammy | ^1.35 | + @grammyjs/auto-retry + @grammyjs/stream |
| **微信通道** | iLink Bot API | 2026.3 协议 | 腾讯官方 HTTP/JSON 协议，非逆向 |
| **浏览器控制** | Chrome DevTools MCP | chrome-devtools-mcp ^0.21 | MCP 协议连接本机 Chrome |
| **MCP 客户端** | @modelcontextprotocol/sdk | ^1.29 | |
| **搜索** | Exa Search API | HTTP | 语义搜索 |
| **网页读取** | Jina Reader | HTTP | `r.jina.ai/{url}` 返回 Markdown |
| **向量 Embedding** | Jina AI v5 / OpenAI text-embedding-3 | HTTP | 自动降级到关键词搜索 |
| **定时任务** | croner | ^9.0 | 纯 TypeScript，支持 cron 表达式 |
| **图表渲染** | @resvg/resvg-js | ^2.6 | SVG→PNG，用于 Telegram 图表回复 |
| **前端框架** | React | ^19.2 | |
| **前端构建** | Vite | ^8.0 | |
| **CSS** | Tailwind CSS | ^4.2 | |
| **UI 组件** | shadcn/ui（基于 @base-ui/react） | ^1.3 | |
| **图标** | @tabler/icons-react | ^3.41 | |
| **状态管理** | Zustand | ^5.0 | |
| **路由** | react-router-dom | ^7.13 | |
| **表单** | react-hook-form + zod | ^7.72 / ^4.3 | |
| **图表（前端）** | ECharts + Recharts | ^6.0 / 2.15.4 | |
| **TypeScript** | typescript | ~5.9 | |

---

## 需要申请的 API Key

| 服务 | 用途 | 申请地址 | 是否必须 |
|------|------|---------|---------|
| **OpenAI API** | 主力 LLM + Embedding | https://platform.openai.com/ | ⚠️ 推荐（默认模型 gpt-4o） |
| **Anthropic API** | Claude 系列模型 | https://console.anthropic.com/ | 可选 |
| **Google AI API** | Gemini 系列模型 | https://aistudio.google.com/ | 可选 |
| **Exa Search API** | 联网语义搜索 | https://exa.ai/ | 可选（联网功能需要） |
| **Jina AI API** | Embedding + 网页读取 | https://jina.ai/ | 可选（有免费额度） |
| **Telegram Bot Token** | Telegram 通道 | https://t.me/BotFather | 可选（需要 Telegram 功能） |
| **微信 iLink Token** | 微信通道 | https://www.wechatbot.dev/ | 可选（需要微信功能） |

> 最低可运行配置：只需要一个 OpenAI API Key。其余全部可选。

---

## 目录结构

```
my-ai-assistant/
├── src/
│   ├── index.ts                 # 入口：初始化 DB、注册通道、启动 Server
│   ├── core/
│   │   ├── agent.ts             # AI Agent 主循环（chat / resetConversation / clearMemory）
│   │   ├── llm.ts               # Vercel AI SDK 封装（streamChat / generateOnce）
│   │   ├── config-manager.ts    # JSON 配置持久化（loadConfig / getConfig / saveConfig）
│   │   ├── memory.ts            # 工作记忆：消息保存、上下文窗口、摘要压缩
│   │   ├── memory-engine.ts     # 四层记忆引擎（buildContext / rememberExplicit / extractFromConversation）
│   │   ├── embedding.ts         # 向量 Embedding 生成与余弦相似度计算
│   │   ├── embedding-catalog.ts # Embedding 模型目录（按 provider 自动选择）
│   │   ├── persona.ts           # 角色管理（getRole / setRole）
│   │   ├── prompt-files.ts      # $USACHI_HOME/*.md prompt 文件读写
│   │   ├── skills.ts            # 工具注册表（initSkills / getAllSkills / registerSkill）
│   │   ├── tool-policy.ts       # OpenClaw 工具权限策略解析器（resolveToolPolicy / expandToolEntries）
│   │   ├── skill-files.ts       # 外部 SKILL.md 发现与加载（$USACHI_HOME/Skills + ~/.agents/skills）
│   │   ├── filesystem-tools.ts  # 文件系统 CRUD 工具（8个一等工具）
│   │   ├── system-tools.ts      # 系统级工具（剪贴板/截图/通知/进程管理）
│   │   ├── tools.ts             # 动态联网工具（webSearch / readWebpage）
│   │   ├── browser.ts           # Chrome DevTools MCP 集成
│   │   ├── model-catalog.ts     # 模型目录（getModelMaxTokens / isReasoningModel）
│   │   ├── scheduler.ts         # 定时任务（croner）
│   │   ├── self-heal.ts         # 自愈看门狗（AI 诊断 + 自动修复）
│   │   ├── health.ts            # 健康检查端点
│   │   ├── parallel.ts          # 多 Agent 并行/扇出
│   │   └── chart-renderer.ts    # ECharts → SVG/PNG 渲染
│   ├── channels/
│   │   ├── adapter.ts           # ChannelAdapter 接口（start/stop/sendMessage）
│   │   ├── telegram.ts          # Telegram Bot 通道（grammy）
│   │   └── wechat.ts            # 微信 iLink Bot 通道
│   ├── server/
│   │   ├── index.ts             # Hono 服务器（密码保护、静态 SPA、API 路由注册）
│   │   ├── onboarding.ts        # 首次引导配置页（独立 Hono 实例）
│   │   └── routes/
│   │       ├── config.ts        # /api/config CRUD
│   │       ├── persona.ts       # /api/persona CRUD
│   │       ├── settings.ts      # /api/settings（只读）
│   │       └── chat.ts          # /api/chat 会话与消息 API（SSE 流式）
│   ├── db/
│   │   ├── client.ts            # Drizzle + libSQL 客户端（initDb / db 导出）
│   │   ├── schema.ts            # 全部表定义（见下方 Schema 章节）
│   │   └── migrate.ts           # 自动迁移脚本
│   ├── types/
│   │   └── config.ts            # UsachiConfig 类型定义 + DEFAULT_CONFIG
│   ├── utils/
│   │   ├── logger.ts            # 结构化日志（写入 bot.log）
│   │   └── runtime-paths.ts     # USACHI_HOME 统一路径解析（resolveRuntimePath）
│   └── scripts/
│       ├── dev.ts               # 开发启动脚本（并行启动 API + Web）
│       └── stop.ts              # 优雅停止脚本
├── web/                         # 前端（独立包）
│   ├── src/
│   │   ├── App.tsx              # 路由定义
│   │   ├── lib/
│   │   │   ├── api.ts           # 所有后端 API 调用（统一封装）+ ToolPolicy 类型 + TOOL_POLICY_PROVIDERS 常量
│   │   │   ├── store.ts         # Zustand 全局状态
│   │   │   └── tool-policy.ts   # 前端纯函数 helper（draft 清洗 / patch 生成 / 回填）
│   │   ├── pages/               # 页面组件（无原生表单元素限制）
│   │   │   ├── ChatPage.tsx
│   │   │   ├── ConfigPage.tsx
│   │   │   ├── PersonasPage.tsx
│   │   │   ├── MemoryPage.tsx
│   │   │   └── SettingsPage.tsx
│   │   └── components/
│   │       ├── Layout.tsx
│   │       ├── Sidebar.tsx
│   │       └── ui/              # shadcn/ui 组件（不直接接 API）
│   └── vite.config.ts           # 代理 /api → 后端 :3000
├── drizzle/                     # 迁移 SQL 文件（自动生成，不手改）
├── drizzle.config.ts
├── package.json
├── tsconfig.json
├── watchdog.ts                  # 独立进程看门狗
├── Dockerfile
└── docker-compose.yml
```

---

## 数据库 Schema

所有表使用 SQLite（libSQL）。数据库文件路径由 `USACHI_HOME` 环境变量决定，默认值：`$USACHI_HOME/usachi.db`（dev 下为 `./data/mybot/usachi.db`，Docker 下为 `/app/data/mybot/usachi.db`）。`dbPath` 配置项存储相对路径（如 `usachi.db`），运行时由 `resolveRuntimePath()` 解析为绝对路径，保证同一份 `config.json` 在宿主机和容器里均可工作。

### 核心表

```typescript
// 对话列表（Web 端）
conversations: { id, agentId, channelUserId, title, createdAt, updatedAt }

// 消息记录（所有通道共用）
messages: { id, agentId, channelType, channelUserId, conversationId, role, content, tokenCount, createdAt }
// role: "user" | "assistant" | "system"
// channelType: "telegram" | "wechat" | "tui" | "web"

// 记忆压缩摘要（短期记忆）
memorySummaries: { id, agentId, channelUserId, summary, messageIdStart, messageIdEnd, createdAt }

// Agent 配置
agents: { id, name, systemPrompt, model, createdAt }
// model 格式: "provider:modelName"，如 "openai:gpt-4o"

// 定时任务
scheduledTasks: { id, name, cronExpr, action(JSON), channelType, channelUserId, enabled, createdAt }
```

### 四层记忆表

```typescript
// 语义事实（三元组，含置信度和激活度衰减）
facts: { id, agentId, channelUserId, category, subject, predicate, object, confidence, activation, source, createdAt, lastAccessedAt }
// category: "preference" | "fact" | "decision" | "relationship"
// source: "extracted" | "explicit" | "inferred"

// 情景记忆（对话摘要，含向量）
episodes: { id, agentId, channelUserId, summary, topics(JSON), mood, messageCount, embedding(JSON float[]), createdAt }

// 用户画像（key-value，自动提取和更新）
userProfile: { id, channelUserId, key, value, confidence, updatedAt }

// 向量索引（语义搜索用）
memoryVectors: { id, sourceType, sourceId, content, embedding(JSON float[]), createdAt }
// sourceType: "fact" | "episode" | "message"
```

---

## 核心模块规格

### 1. 配置管理（config-manager.ts）

配置文件路径：`$USACHI_HOME/config.json`

- `USACHI_HOME` 环境变量：dev 默认 `./data/mybot`，Docker 设为 `/app/data/mybot`
- 同一份 `config.json` 和 `usachi.db` 被宿主机 dev 进程与容器共享，无需两份配置

配置分区：

```typescript
interface UsachiConfig {
  general: {
    botName: string           // 默认 "乌萨奇"
    defaultLanguage: string   // "zh-CN"
    defaultModel: string      // "openai:gpt-4o"
    systemPrompt: string
    dbPath: string            // 相对路径，默认 "usachi.db"；运行时由 resolveRuntimePath() 解析
  }
  role: { name, avatar, systemPrompt }
  models: { openai, anthropic, google: boolean; temperature, maxTokens: number }
  channels: {
    telegram: { enabled }
    wechat: { enabled, pollIntervalMs, autoReLogin }
    tui: { enabled }
  }
  memory: { workingMemoryLimit, summaryThreshold, dailyDecayRate, semanticSearchThreshold, embeddingModel, maxFactsPerRetrieval }
  selfHealing: { enabled, checkIntervalMs, maxAttempts }
  webPanel: { port, passwordEnabled, password, authSecret }
  tools: {
    webSearch: { enabled, maxResults }
    webReader: { enabled, maxContentLength }
    browser: { enabled, args: string[] }
    localAccess: {
      enabled: boolean        // 本地工具总开关，默认 true
      adminOnly: boolean      // 仅 adminChatId 用户可用，默认 false
      filesystem: boolean     // 文件系统工具，默认 true
      shell: boolean          // Shell 命令，默认 true
      openApp: boolean        // 打开应用，默认 true
      system: boolean         // 系统工具（剪贴板/截图等），默认 true
    }
    policy?: {                // OpenClaw 风格工具权限策略（见 Section 4）
      profile?: "minimal" | "chat" | "coding" | "full"
      allow?: string[]        // 显式加入工具名或 group:xxx
      deny?: string[]         // 显式排除，优先于 allow
      byProvider?: {          // 按 provider 覆盖
        [provider: string]: { profile?, allow?, deny? }
      }
      byModel?: {             // 按具体模型覆盖（优先级最高）
        [modelId: string]: { profile?, allow?, deny? }
      }
    }
  }
  credentials: {
    telegramBotToken, openaiApiKey, openaiBaseUrl,
    anthropicApiKey, anthropicBaseUrl, googleApiKey, googleBaseUrl,
    exaApiKey, jinaApiKey, wechatIlinkToken, adminChatId,
    openaiEmbeddingApiKey, openaiEmbeddingBaseUrl, openaiEmbeddingModel
  }
}
```

关键约定：
- 配置持久化到 JSON，环境变量仅用于首次初始化/重置时的默认值填充
- 所有模块调用 `getConfig()`（同步）获取当前配置，不直接读文件
- 初始化流程：`loadConfig()` → 深度合并 DEFAULT_CONFIG → `normalizeToolPolicy()` → 写回文件

### 2. Agent 主循环（agent.ts）

`chat()` 函数流程：
1. 检测显式记忆指令（"记住" / "记一下" 等关键词）
2. 保存用户消息到 DB
3. `buildContext()` — 用四层记忆构建上下文窗口
4. `streamChat()` — 流式调用 LLM，传入当前可用工具集
5. 保存 AI 回复到 DB
6. 异步：摘要压缩 + 记忆提取（不阻塞响应）

模型 ID 格式：`"provider:modelName"`，如 `"openai:gpt-4o"`、`"anthropic:claude-opus-4-5"`

### 3. 四层记忆系统（memory-engine.ts）

```
工作记忆  → 最近 N 条消息（实时，消息表）
短期记忆  → 摘要压缩（超过阈值时 LLM 自动压缩，memorySummaries 表）
长期记忆  → 语义事实三元组（facts 表）+ 情景摘要（episodes 表）
永久记忆  → 用户画像 key-value（userProfile 表）
```

检索流程（buildContext）：
1. 取最近 N 条工作记忆
2. 语义检索：用当前 query 向量搜索 facts + episodes
3. 激活度衰减：每日 5% 衰减，访问时重置
4. 用户画像：追加到 system prompt 开头

记忆提取：每轮对话结束后，异步调用 LLM 对最近 10 条消息做**一次结构化提取**（JSON schema 约束输出），同时产出：
- 新的事实三元组 → facts 表
- 对话情景摘要 → episodes 表
- 用户画像更新 → userProfile 表

> 注：提取使用当前对话的同一模型（`getEffectiveModel()`），避免硬编码 provider 导致的额外计费。

### 4. 工具权限模型（skills.ts + tool-policy.ts）

工具暴露由 `tools.policy` 配置驱动，采用 **OpenClaw 三层解析**模型，而非按消息内容猜测工具。

#### 签名

```typescript
getAllSkills(opts: {
  channelType?: string
  channelUserId?: string
  modelId?: string           // 实际使用的模型 ID（provider:modelName）
}): Tool[]
```

#### 解析顺序（优先级由低到高）

```
1. global profile（tools.policy.profile）
      ↓  覆盖
2. byProvider（tools.policy.byProvider[provider]）
      ↓  覆盖
3. byModel（tools.policy.byModel[modelId]）
```

每层均可包含 `profile / allow / deny` 三个字段，高层覆盖低层。同一层内 **`deny` 优先于 `allow`**。

#### 内置 profile 对应工具集

| profile | 包含工具 |
|---------|---------|
| `minimal` | getCurrentTime、calculate |
| `chat` | minimal + dataviz、generateComponent、webSearch、readWebpage |
| `coding` | chat + filesystem、runShellCommand、openApplication |
| `full` | coding + system（剪贴板/截图等）+ browser |

默认 profile：`coding`（`DEFAULT_TOOL_POLICY`）。

#### localAccess 最终硬限制

`tools.localAccess` 在策略解析后作**最后一道硬限制**：
- `localAccess.enabled = false` → 无论策略如何，移除所有本地工具
- `localAccess.filesystem = false` → 无论策略 allow，移除文件系统工具
- 其余子开关同理

即：策略控制"想开放什么"，`localAccess` 控制"物理上允许什么"。

#### 通道权限

- `web` / `tui` 通道 → 视为管理员，`localAccess.adminOnly` 不适用
- `telegram` / `wechat` 通道 + `adminOnly=true` → 需 `channelUserId === adminChatId`
- 缺失 channelType/channelUserId 时**不默认放行**（安全收紧）

#### 配置示例

```json
"tools": {
  "policy": {
    "profile": "chat",
    "byModel": {
      "anthropic:claude-sonnet-4-5": { "profile": "coding" },
      "anthropic:claude-haiku-4-5": { "profile": "minimal" }
    }
  }
}
```

> 上例中 Haiku 只暴露 `minimal`（2 个工具），Sonnet 暴露 `coding`，其余模型暴露 `chat`。有效控制 Haiku 的 token 开销。

### 5. 工具清单

#### 文件系统（filesystem-tools.ts）

| 工具名 | 参数 | 说明 |
|--------|------|------|
| `listDirectory` | path, recursive?, maxDepth? | 列出目录，含大小/时间元数据 |
| `readFile` | path, startLine?, endLine?, encoding? | 读取文件，支持行范围，最大 512KB |
| `writeFile` | path, content, append?, createDirs? | 写/追加文件，自动创建父目录 |
| `editFile` | path, old_str, new_str | 精确查找替换（唯一匹配约束） |
| `deleteFile` | path, recursive? | 删除文件/目录 |
| `moveFile` | source, destination | 移动/重命名 |
| `copyFile` | source, destination, recursive? | 复制 |
| `fileInfo` | path | 获取元数据（大小/权限/时间戳） |

#### 系统工具（system-tools.ts）

| 工具名 | 参数 | 说明 |
|--------|------|------|
| `clipboardRead` | — | 读取系统剪贴板 |
| `clipboardWrite` | content | 写入系统剪贴板 |
| `screenshot` | savePath?, region? | 截图（macOS screencapture / Linux scrot） |
| `notify` | title, message, sound? | 发送桌面通知 |
| `listProcesses` | filter?, limit? | 列出进程（含 PID/CPU/内存） |
| `killProcess` | pid, signal? | 按 PID 终止进程 |

#### Shell 工具（skills.ts）

| 工具名 | 参数 | 说明 |
|--------|------|------|
| `runShellCommand` | command, cwd?, timeoutMs? | 执行 shell 命令，默认超时 30s，最大 5min |
| `openApplication` | application?, target?, args? | 跨平台打开应用/文件/URL |

#### 联网工具（tools.ts）

| 工具名 | 参数 | 说明 |
|--------|------|------|
| `webSearch` | query, numResults? | Exa 语义搜索，需 Exa API Key |
| `readWebpage` | url, maxLength? | Jina Reader，返回 Markdown |

### 6. 通道规格

#### Telegram（telegram.ts）

- 依赖：grammy + @grammyjs/auto-retry + @grammyjs/stream
- 流式回复：使用 grammy stream 插件，打字机效果
- ECharts 图表：检测回复中的 ` ```echarts ` 块，渲染为 PNG 发送
- 工具状态回调：工具调用时通过 `onStatusChange` 发送"乌萨奇正在..."状态消息

#### 微信 iLink Bot API（wechat.ts）

- 基座：`https://ilinkai.weixin.qq.com`
- 登录：自动获取 QR 码链接 → 用户扫码 → 获得 bot_token
- 轮询：35s 长轮询（get_updates_buf 断点续传）
- 持久化：`$USACHI_HOME/wechat/session.json` + `contacts.json` + `sync-buf.txt`
- 错误恢复：errcode -14 → 自动重新扫码

### 7. Web 服务器（server/index.ts）

- 框架：Hono v4
- 端口：默认 3000（可配置）
- 密码保护：可选，JWT token 认证
- 静态资源：生产环境从 `web/dist/` 提供 SPA
- SSE 流式：`/api/chat/conversations/:id/messages` 返回 Server-Sent Events

API 路由：

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/config` | 完整配置（credentials 脱敏） |
| PATCH | `/api/config` | 部分更新配置 |
| POST | `/api/config/reset` | 重置为默认配置 |
| GET | `/api/config/credentials` | credentials 是否已配置（布尔值） |
| POST | `/api/auth/login` | 密码登录 |
| POST | `/api/auth/logout` | 登出 |
| GET | `/health` | 健康检查 `{ status: "ok" }` |
| GET | `/api/chat/conversations` | 会话列表 |
| POST | `/api/chat/conversations` | 新建会话 |
| GET | `/api/chat/conversations/:id/messages` | 消息列表 |
| POST | `/api/chat/conversations/:id/messages` | 发消息（SSE 流式） |
| GET | `/api/persona` | 当前角色 |
| PUT | `/api/persona` | 更新角色 |

### 8. 前端架构（web/src）

分层规则：
- `pages/`：页面级组件，使用 shadcn/ui 表单组件（不用原生 `<form>`/`<input>`）
- `components/ui/`：shadcn/ui 组件，不直接调用 `lib/api`
- `lib/api.ts`：所有后端请求统一封装（不在页面内直接 fetch）
- `lib/store.ts`：Zustand 全局状态，不引入 React
- `lib/`：不引入 React

状态管理（Zustand store）：
- `config`：当前配置状态
- `conversations`：会话列表
- `messages`：当前会话消息
- `persona`：当前角色
- `theme`：深色/浅色主题

### 9. 自愈系统（self-heal.ts）

触发条件：健康检查 `/health` 失败

流程：
1. 读取最近 50 条日志（bot.log）+ 当前配置（脱敏）
2. 调用 LLM 诊断问题，输出 JSON：`{ action, detail, command? }`
3. 执行修复：
   - `restart`：通知管理员（Telegram）
   - `fix_config`：通知管理员说明配置问题
   - `run_command`：执行修复命令（`sh -c command`）
   - `escalate`：通知管理员人工介入

### 10. Skills 系统（skill-files.ts）

外部 Skill 加载路径（优先级：project > global）：
- 项目级：`~/mybot/Skills/`
- 全局：`~/.agents/skills/`（其次 `~/.codex/skills/`）

Skill 格式：每个 skill 是一个目录，包含 `SKILL.md`，文件头部有 frontmatter：
```yaml
---
name: skill-name
description: 技能的简短描述
---
技能的完整指令内容...
```

安装方式：
- 从 skills.sh URL 安装：`https://skills.sh/{owner}/{repo}/{skill-name}`
- 从 GitHub SKILL.md URL 安装

运行时流程：
1. 启动时扫描所有 skill 目录，只读取 frontmatter（名字+描述）
2. 将摘要注入 system prompt
3. LLM 需要某个 skill 时，调用 `loadSkill` 工具读取完整内容

---

## 启动流程

```
src/index.ts
  ├── initDb()           → 初始化 SQLite，执行未应用的迁移
  ├── initSkills()       → 注册内置工具（runShellCommand / filesystem / system / dataviz...）
  ├── initBrowser()      → 连接 Chrome DevTools MCP（如果 browser.enabled）
  ├── startServer()      → 启动 Hono（端口 3000）
  ├── startChannels()    → 启动已启用的通道（Telegram / WeChat / TUI）
  └── startScheduler()   → 启动定时任务
```

---

## 配置文件位置

```
$USACHI_HOME/              # 默认：dev → ./data/mybot，Docker → /app/data/mybot
├── config.json              # 主配置文件（程序自动管理）
├── usachi.db                # SQLite 数据库
├── AGENTS.md                # 全局 system prompt（覆盖配置中的 systemPrompt）
├── SOUL.md                  # 角色定义 prompt
├── Skills/                  # 项目级 Skills 目录
└── wechat/
    ├── session.json
    ├── contacts.json
    └── sync-buf.txt
```

---

## 开发命令

```bash
# 安装依赖
bun install

# 开发模式（同时启动后端 + 前端热重载）
bun run dev

# 仅后端
bun --watch src/index.ts

# 仅前端
cd web && npx vite

# 数据库迁移（schema 变更后运行）
bun run db:generate
bun run db:migrate

# 构建生产包（单二进制 + 前端静态）
bun run build

# 运行看门狗（独立进程）
bun run watchdog
```

---

## 环境变量（仅用于首次初始化，运行时以 JSON 配置为准）

```bash
# 运行目录（决定 config.json / usachi.db / Skills/ 等所有数据的存储位置）
USACHI_HOME=./data/mybot           # dev 默认值；Docker 设为 /app/data/mybot

OPENAI_API_KEY=sk-...
OPENAI_BASE_URL=https://api.openai.com/v1      # 可替换为代理
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_API_KEY=...
EXA_API_KEY=...
JINA_API_KEY=jina_...
TELEGRAM_BOT_TOKEN=...
ADMIN_CHAT_ID=...                              # Telegram 管理员 Chat ID
DB_PATH=                                       # 覆盖数据库路径（仅首次初始化）
```

---

## Docker 运行

```yaml
# docker-compose.yml
services:
  bot:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - ./data:/app/data      # 宿主机 ./data 挂载到容器 /app/data，与 dev 共享同一份数据
    environment:
      - USACHI_HOME=/app/data/mybot
      - OPENAI_API_KEY=${OPENAI_API_KEY}
```

> dev 与 Docker 使用同一份 `./data/mybot/config.json` 和 `usachi.db`，无需维护两套配置。

---

## 给 AI 的构建指令模板

把以下内容连同本文档一起发给 AI：

```
请根据上面的架构蓝图，帮我从零搭建这个个人 AI 助手项目。

具体要求：
1. 使用 TypeScript + Bun 运行时
2. 严格按照目录结构创建所有文件
3. 数据库 schema 完全按规格实现，使用 Drizzle ORM
4. 配置系统使用 JSON 持久化，不依赖环境变量运行
5. 四层记忆系统完整实现（工作/摘要/语义/画像）
6. 工具权限模型使用 `tools.policy` 配置驱动（OpenClaw 三层解析：global profile → byProvider → byModel，deny 优先，localAccess 最终硬限制）
7. 前端使用 React 19 + Vite + Tailwind CSS v4
8. 先生成 package.json 和 tsconfig.json
9. 然后按模块顺序：db → types → config-manager → memory → agent → skills → channels → server → web
```
