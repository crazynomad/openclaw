# OpenClaw 深度解析 | 视频脚本 V2

> 🎬 时长：约 40 分钟
> 🎯 目标：不只是介绍功能，而是**揭秘设计背后的 Why**
> 💡 风格：热点切入 → 硬核拆解 → 哲学升华

---

## 开场 Hook（0:00 - 2:00）

**【画面：各种 AI 聊天界面快速切换】**

你有没有想过，为什么 ChatGPT 只能在网页里等你问它？

为什么它不能主动提醒你："嘿，你昨天说要给客户回邮件，还没做呢"？

为什么你得打开 5 个不同的 App，才能和不同平台的朋友聊天？

**【画面：展示 OpenClaw】**

今天我要带你看一个开源项目，它试图解决这些问题。

但更重要的是，我想带你**深入源代码**，看看这些功能背后的设计智慧。

这不是一个"功能介绍"视频。这是一个**技术拆解**视频。

准备好了吗？

---

## 第一幕：它为什么"活着"？（2:00 - 12:00）

### 问题：AI 太被动了

**【画面：用户等待 AI 回复的场景】**

传统的 AI 聊天有一个根本性的问题：

**它是被动的。**

你问，它答。你不问，它就沉默。

但真正的"助手"不应该是这样。一个好的助手应该：

- 主动提醒你待办事项
- 在合适的时间联系你
- 替你关注一些事情

OpenClaw 是怎么做到的？

### 揭秘：Heartbeat（心跳机制）

**【画面：心脏跳动的动画，叠加代码】**

答案是一个叫 **Heartbeat** 的机制。

让我打开源代码给你看。

```typescript
// src/infra/heartbeat-runner.ts

// 默认每 30 分钟运行一次
const DEFAULT_HEARTBEAT_EVERY = "30m";

// 心跳提示词
const HEARTBEAT_PROMPT =
  "Read HEARTBEAT.md if it exists. Follow it strictly. " +
  "If nothing needs attention, reply HEARTBEAT_OK.";
```

**【画面：流程图动画】**

这是它的工作流程：

```
每 30 分钟
    │
    ▼
┌───────────────────────────────┐
│  读取 HEARTBEAT.md（待办清单）  │
└───────────────────────────────┘
    │
    ▼
┌───────────────────────────────┐
│  调用 AI："有什么需要关注的吗？" │
└───────────────────────────────┘
    │
    ├─── AI 说 "HEARTBEAT_OK" ─── 静默丢弃，不打扰用户
    │
    └─── AI 说有事要报告 ──────── 发送消息给用户
```

**【稍作停顿】**

注意这个设计的精妙之处：

它不是简单地每 30 分钟发一条消息骚扰你。而是：

1. **先问 AI**："有没有什么需要告诉用户的？"
2. **AI 判断**：如果没事，回复 `HEARTBEAT_OK`
3. **系统过滤**：看到 `HEARTBEAT_OK`，就**不发消息**

只有当 AI 认为**真的有事**的时候，你才会收到消息。

### HEARTBEAT.md：AI 的"待办清单"

**【画面：展示一个 HEARTBEAT.md 文件】**

那 AI 怎么知道要关注什么？

答案是一个 Markdown 文件：`HEARTBEAT.md`

```markdown
# 心跳检查清单

- 检查是否有未回复的重要消息
- 如果是白天，可以问候一下用户
- 如果有任务被阻塞，记录下来等下次询问
```

**【画面：AI 读取文件的动画】**

每次心跳，AI 都会读取这个文件，然后根据清单检查各种事项。

这意味着什么？

**你可以通过修改一个文本文件，来改变 AI 的行为。**

不需要写代码，不需要改配置，就是一个普通的 Markdown 文件。

### Active Hours：尊重你的睡眠

**【画面：时钟从深夜转到白天】**

但如果心跳是每 30 分钟运行一次，凌晨 3 点也会运行吗？

不会。

```json
{
  "heartbeat": {
    "every": "30m",
    "activeHours": {
      "start": "08:00",
      "end": "24:00"
    }
  }
}
```

你可以设置 **Active Hours**。在这个时间范围外，心跳会自动暂停。

**【稍作停顿】**

这些细节加在一起，才让 AI 从"工具"变成了"助手"。

它不只是在等你问问题。它在**主动关注你的生活**，但又**尊重你的边界**。

---

## 第二幕：它为什么不会"发疯"？（12:00 - 22:00）

### 问题：并发是 AI 应用的噩梦

**【画面：多个请求同时涌入的动画】**

当你有多个消息同时进来，会发生什么？

- WhatsApp 来了一条消息
- 同时 Telegram 也来了一条
- 心跳也恰好触发了
- 还有一个定时任务要执行

如果这些任务同时执行，会怎样？

**【画面：AI 逻辑混乱的示意】**

最坏的情况：**AI 会把不同对话的上下文混在一起。**

你在 WhatsApp 上问的问题，可能收到 Telegram 对话的回答。

这在 AI 应用中是一个常见的 bug，叫做 **Race Condition（竞态条件）**。

### 揭秘：Lane-based Queue（车道队列）

**【画面：高速公路多车道的动画】**

OpenClaw 的解决方案非常优雅。让我给你看代码。

```typescript
// src/process/lanes.ts

export const enum CommandLane {
  Main = "main",        // 主对话车道
  Cron = "cron",        // 定时任务车道
  Subagent = "subagent", // 子 Agent 车道
  Nested = "nested",     // 嵌套调用车道
}
```

它把不同类型的任务分到**不同的"车道"**里。

**【画面：交通指挥的动画】**

```
用户消息 ────→ [Main 车道] ───┐
                              │
定时任务 ────→ [Cron 车道] ───┼───→ Agent
                              │
心跳检查 ────→ [Main 车道] ───┘

每个车道内部：串行执行（排队）
不同车道之间：可以并行
```

关键设计：

1. **同一车道内**：任务**排队执行**（串行）
2. **不同车道间**：可以**并行执行**
3. **每个车道**有独立的**并发上限**

```typescript
// src/gateway/server-lanes.ts

setCommandLaneConcurrency(CommandLane.Cron, cfg.cron?.maxConcurrentRuns ?? 1);
setCommandLaneConcurrency(CommandLane.Main, resolveAgentMaxConcurrent(cfg));
```

**【稍作停顿】**

为什么这个设计有效？

**因为它把"串行"和"并行"放在了正确的层次。**

同一个用户的消息需要串行——保证上下文连贯。
不同类型的任务可以并行——提高整体效率。

这是一个在软件工程中反复出现的模式：**在正确的粒度上加锁**。

### Session 隔离：每个对话是独立的

**【画面：Session Key 的结构】**

除了车道，OpenClaw 还有另一层隔离：**Session**。

```
Session Key 格式：

agent:main:whatsapp:+8613800138000
│      │      │           │
│      │      │           └── 用户标识
│      │      └── 来自哪个平台
│      └── 用哪个 Agent
└── 前缀
```

同一个用户在 WhatsApp 和 Telegram 上的对话，是**完全独立的 Session**。

这意味着：
- 上下文不会混淆
- 可以有不同的对话历史
- 可以有不同的"人设"

**【稍作停顿】**

这两个机制——车道和 Session——共同保证了：

**即使有 100 个人同时给你发消息，AI 也不会"发疯"。**

---

## 第三幕：它是怎么"连接世界"的？（22:00 - 32:00）

### 问题：每个平台都是一座孤岛

**【画面：各个聊天平台被海水隔开】**

WhatsApp、Telegram、Discord、Slack……

每个平台都有自己的：
- 消息格式
- API 接口
- 认证方式
- 限制条件

如果你想写一个程序连接所有这些平台，你得学习每一个 API。

### 揭秘：Channel Adapter（通道适配器）

**【画面：翻译官的比喻】**

OpenClaw 的解决方案是：**一个翻译官**。

```
WhatsApp (德语) ──┐                    ┌── WhatsApp (德语)
                  │                    │
Telegram (法语) ──┼──→ 翻译官 ←──────┼── Telegram (法语)
                  │   (统一格式)       │
Discord (日语)  ──┘                    └── Discord (日语)
```

让我给你看真实的代码。

**【画面：代码展示】**

WhatsApp 的原始消息：

```javascript
{
  key: {
    remoteJid: "8613800138000@s.whatsapp.net",
    fromMe: false,
    id: "3EB0A0B3C..."
  },
  message: {
    conversation: "你好"
  }
}
```

Telegram 的原始消息：

```javascript
{
  message_id: 123,
  from: { id: 456789, first_name: "小明" },
  chat: { id: 456789, type: "private" },
  text: "你好"
}
```

**【画面：转换动画】**

经过 Channel Adapter 翻译后，变成统一格式：

```javascript
{
  channel: "whatsapp",  // 或 "telegram"
  sender: "+8613800138000",  // 统一的发送者标识
  text: "你好",
  isGroup: false,
  timestamp: 1699999999
}
```

**【稍作停顿】**

这就是**适配器模式（Adapter Pattern）**的实际应用。

核心代码只需要处理这个统一格式。不管外面是什么平台，进到系统里都是同一种"语言"。

### Channel 插件：无限扩展

**【画面：插件架构图】**

更妙的是，每个 Channel 都是一个**插件**。

```
extensions/
├── discord/     ← Discord 插件
├── telegram/    ← Telegram 插件
├── whatsapp/    ← WhatsApp 插件
├── slack/       ← Slack 插件
├── signal/      ← Signal 插件
├── msteams/     ← MS Teams 插件
├── matrix/      ← Matrix 插件
└── ...
```

想支持一个新平台？写一个插件就行。

核心代码**一行都不用改**。

这就是**开闭原则（Open-Closed Principle）**：

> 对扩展开放，对修改关闭。

---

## 第四幕：记忆的秘密（32:00 - 38:00）

### 问题：AI 是金鱼

**【画面：金鱼游泳】**

大多数 AI 聊天应用有一个尴尬的问题：

**AI 记不住东西。**

或者说，它的"记忆"仅限于当前的对话上下文。

### 揭秘：混合记忆系统

**【画面：记忆系统架构图】**

OpenClaw 的记忆系统有三层：

```
┌─────────────────────────────────────────────────┐
│  Layer 1: Session（对话记忆）                    │
│  存储在 JSON 文件里，每个对话独立                │
└─────────────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────────────┐
│  Layer 2: Workspace Files（工作区文件）          │
│  AGENTS.md, HEARTBEAT.md, 任何 Markdown 文件     │
└─────────────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────────────┐
│  Layer 3: Vector Memory（向量记忆）              │
│  SQLite + sqlite-vec，支持语义搜索              │
└─────────────────────────────────────────────────┘
```

**【画面：代码展示】**

```typescript
// src/memory/manager.ts

// 混合搜索：关键词 + 语义
import { bm25RankToScore, buildFtsQuery, mergeHybridResults } from "./hybrid.js";

// Markdown 分块
import { chunkMarkdown } from "./internal.js";

// 向量搜索
import { searchVector } from "./manager-search.js";
```

让我解释一下这个设计：

**Layer 1: Session** —— 短期记忆
- 当前对话的内容
- 存在 `~/.openclaw/sessions/` 目录
- JSON 格式，简单可靠

**Layer 2: Workspace Files** —— 工作记忆
- `AGENTS.md`：Agent 的"人设"
- `HEARTBEAT.md`：心跳检查清单
- 其他 Markdown 文件：任何你想让 AI 知道的信息

**Layer 3: Vector Memory** —— 长期记忆
- 把 Markdown 文件切成小块
- 生成向量嵌入（Embedding）
- 存入 SQLite（使用 sqlite-vec 扩展）
- 支持语义搜索

**【稍作停顿】**

最有趣的是**混合搜索**。

```typescript
// 不只是关键词匹配
// 也不只是向量搜索
// 而是两者结合

const results = mergeHybridResults(
  keywordResults,  // BM25 关键词排序
  vectorResults    // 向量语义相似度
);
```

为什么要混合？

- **关键词搜索**擅长精确匹配（"iPhone 15"）
- **向量搜索**擅长语义理解（"苹果最新手机"）

两者结合，才能真正理解你的问题。

---

## 尾声：代码之上的思考（38:00 - 40:00）

**【画面：从代码缩小到星空】**

我们今天看了很多代码。

心跳机制、车道队列、通道适配器、混合记忆……

但我想请你思考一个更大的问题：

**这些代码为什么会这样设计？**

**【稍作停顿】**

每一个设计决策背后，都是对某个**问题**的回应。

- 心跳机制 → 解决"AI 太被动"的问题
- 车道队列 → 解决"并发会出错"的问题
- 通道适配器 → 解决"平台太分散"的问题
- 混合记忆 → 解决"AI 是金鱼"的问题

**【面对镜头】**

作为开发者，我们最重要的能力不是写代码。

是**理解问题**，然后**设计解决方案**。

代码只是把解决方案写下来而已。

**【画面：GitHub Star 数在增长】**

OpenClaw 是开源的。

如果你对这些设计感兴趣，去 GitHub 上看看源代码。

不只是看**怎么写**，更要想**为什么这么写**。

这才是真正的学习。

感谢观看。我们下期再见。

---

## 📝 导演笔记

### 画面节奏

| 时间段 | 类型 | 说明 |
|--------|------|------|
| 0:00-2:00 | 真人 + 快剪 | Hook，抓注意力 |
| 2:00-12:00 | 代码 + 动画 | 心跳机制拆解，重点是流程图动画 |
| 12:00-22:00 | 动画为主 | 车道比喻用交通动画，强调"串行 vs 并行" |
| 22:00-32:00 | 代码 + 翻译官动画 | 适配器模式，before/after 对比 |
| 32:00-38:00 | 架构图 + 代码 | 三层记忆，混合搜索 |
| 38:00-40:00 | 真人面对镜头 | 哲学升华，安静有力 |

### 关键 Take-away

1. **Heartbeat 不是定时发消息，而是"定时问 AI 要不要发消息"**
2. **Lane 是在正确粒度上加锁的典范**
3. **适配器模式让"一次编写，到处运行"成为可能**
4. **混合搜索 = 关键词 + 语义，两者互补**

### 与 V1 的区别

| V1 | V2 |
|----|----|
| 介绍功能 | 拆解源码 |
| 3 个洞见 | 4 个机制 |
| 比喻为主 | 代码+比喻 |
| 宏观架构 | 微观实现 |

**V2 更硬核，但保留了可理解性。**

---

*"The purpose of abstraction is not to be vague, but to create a new semantic level in which one can be absolutely precise." — Edsger Dijkstra*
