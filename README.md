# 个人 AI 助手架构蓝图 · Personal AI Assistant Blueprint

> 把 `BLUEPRINT.md` 完整粘贴给任意代码 AI（Claude / GPT / Gemini），AI 就能帮你把完整项目搭起来。

## 这是什么

一份**可以直接喂给 AI 的架构规格文档**，覆盖：

- 完整技术选型（精确版本号）
- 每个模块的职责边界和接口定义
- 数据库 schema 设计（SQLite + Drizzle ORM）
- 工具权限模型（OpenClaw 级本地 agent 能力）
- 需要哪些 API Key，每个去哪里申请
- 四层记忆系统设计
- 启动流程 + 开发命令

## 用法

```bash
# 把 BLUEPRINT.md 内容完整粘贴给 Claude 或 GPT，然后说：
# "请按这份规格，帮我从零搭建这个个人 AI 助手项目。"
cat BLUEPRINT.md | pbcopy   # macOS 复制到剪贴板
```

## 你会得到什么

一个运行在本机的个人 AI 助手，具备：

| 能力 | 说明 |
|------|------|
| **多模型支持** | OpenAI / Anthropic / Google，一行切换 |
| **四层记忆** | 工作记忆 → 摘要压缩 → 语义事实/情景 → 用户画像，向量搜索 |
| **多通道** | Telegram Bot + 微信 iLink Bot + Web Dashboard |
| **本地 Agent** | 文件系统 CRUD / Shell / 系统工具 / 浏览器控制，权限配置驱动 |
| **控制面板** | Web UI：配置 / 对话 / 记忆管理 / Token 用量 |
| **自愈看门狗** | AI 诊断 + 自动修复，健康检查失败时告警 |

## 需要哪些 API Key

| 服务 | 是否必须 | 申请地址 |
|------|---------|---------|
| OpenAI API | ⚠️ 推荐 | https://platform.openai.com/ |
| Anthropic API | 可选 | https://console.anthropic.com/ |
| Google AI API | 可选 | https://aistudio.google.com/ |
| Exa Search API | 可选（联网搜索） | https://exa.ai/ |
| Jina AI API | 可选（Embedding + 读网页） | https://jina.ai/ |
| Telegram Bot Token | 可选（Telegram 通道） | https://t.me/BotFather |
| 微信 iLink Token | 可选（微信通道） | https://www.wechatbot.dev/ |

> 最低启动：只需要一个 OpenAI API Key。

## 文件说明

- **`BLUEPRINT.md`** — 完整架构规格，喂给 AI 用的
- **`README.md`** — 本文件

## 相关链接

- 作者项目代码：[github.com/eachann1024/mybot](https://github.com/eachann1024/mybot)
- 文章：[2026年，自己造一个AI助手才是正确答案](#)
