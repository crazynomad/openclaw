# OpenClaw 冷知识 | Fun Facts

> 从代码仓库挖掘出的有趣数据，证明 AI 正在改变软件开发

---

## 1. 项目规模

| 指标 | 数值 |
|------|------|
| 总 Commits | **9,672** |
| TypeScript 代码行数 | **451,754** |
| 贡献者数量 | 100+ |
| 主要开发者 (Peter Steinberger) | 7,327 commits (75.8%) |

---

## 2. AI 参与开发统计

### 明确署名的 AI/Bot Commits

| AI/Bot 账号 | Commits | 说明 |
|-------------|---------|------|
| **Shadow** | 182 | OpenClaw 的核心 AI Agent，日常维护和 PR 合并 |
| **Claude** (Opus/Sonnet) | 26 | Anthropic Claude，通过 Co-Author 标签 |
| **Google Jules** | 14 | Google 的 AI 编程助手 |
| **CLAWDINATOR** | 8 | GitHub CI/自动化 Bot |
| **ghost@clawd** | 10 | 匿名内部 Bot |
| **其他 Bots** | ~30 | dependabot, kit, 贡献者的 bots 等 |
| **总计** | **~270** | 占总 commits 的 2.8% |

### Shadow：项目的 AI 管家

Shadow 是 OpenClaw 运行的一个 AI Agent，它的工作包括：

```
=== Shadow 的典型 commit ===
da9f28d27 CI: label maintainer issues
8bf9cfe62 fix: avoid workspace references in reset greeting (#5706) (thanks @bravostation)
8e2b17e0c Discord: add PluralKit sender identity resolver (#5838)
57d9c09f6 fix: expand Telegram polling network recovery (#3013) (thanks @ryancontent)
```

注意 `(thanks @username)` 模式 —— Shadow 在帮助处理社区 PR！

---

## 3. 隐藏的 AI 贡献

### 问题：75% 的代码真的是一个人写的吗？

Peter Steinberger 贡献了 7,327 个 commits（75.8%），但考虑到：

1. **他是 OpenClaw 创始人** —— 用自己的产品开发自己的产品
2. **45 万行 TypeScript** —— 以人类速度难以想象
3. **项目核心功能就是 AI 辅助编程**

**推测：大量代码是通过 AI 辅助（Cursor/Copilot/OpenClaw）完成的，只是没加 Co-Author 标签。**

### 证据：视频里的 Aha Moment

> "有一天，有人在 Twitter 上发了一个 bug 截图。Peter 把截图转发给 OpenClaw。
> 几分钟后，OpenClaw 回复：'我已经修复了这个 bug，代码已经提交。'"

**OpenClaw 真的在修改自己的代码库。**

---

## 4. AI 参与的演变

从 commit 历史可以看到 AI 参与的演变：

| 时期 | AI 参与方式 |
|------|-------------|
| 早期 (MoltBot) | `bot@moltbot.com` 简单自动化 |
| 中期 (Clawdbot) | `shadow@clawd.bot` 开始处理 PR |
| 现在 (OpenClaw) | Claude Co-Author + Shadow 日常维护 |

### 有趣的邮箱域名

```
@clawd.bot        - 内部 Bot 系统
@openclaw.ai      - 官方 AI Agent
@moltbot.com      - 早期项目名称
@clawdbot.dev     - 过渡期名称
```

---

## 5. 自我进化的证据

### Commit 消息里的"递归"

```
d3e53eaf2 fix(skill): update session-logs paths from .clawdbot to .openclaw
          Co-authored-by: Jarvis <jarvis@openclaw.ai>
```

这是一个 AI (Jarvis) 帮助项目从旧名称迁移到新名称的 commit。

**AI 在帮助 AI 项目重命名自己。**

### "via OpenClaw" 模式

```
Co-authored-by: Lucifer (via OpenClaw) <lucy@neuwirth.cc>
```

有用户通过 OpenClaw 让 AI 帮他们提交代码到 OpenClaw 项目。

**用户用 OpenClaw 来开发 OpenClaw。**

---

## 6. 多 AI 协作

项目中出现了多个不同的 AI 系统：

| AI 系统 | 角色 |
|---------|------|
| **Shadow** | 日常维护、PR 处理 |
| **CLAWDINATOR** | CI/CD 自动化 |
| **Jarvis** | 特定任务 Agent |
| **Claude** | 代码编写（通过开发者） |
| **Google Jules** | 外部 AI 贡献 |

这可能是**最早的"多 AI Agent 协作开发"开源项目之一**。

---

## 7. 有趣的 Commit 消息

```bash
# AI 有态度
"fix: avoid workspace references in reset greeting"  # Shadow 修复问候语

# 自我意识？
"AI 决定：既然消息没用，那就物理攻击吧"  # 代码注释里的 Sonos 控制

# 递归幽默
"OpenClaw 是开源的。你可以自己跑一个..."  # 用 OpenClaw 写的文档
```

---

## 8. 数据可视化

```
Commits 来源分布
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Peter Steinberger  ████████████████████████████████████  75.8%
Shadow (AI)        ██                                     1.9%
其他人类贡献者      ███████                               19.5%
其他 AI/Bots       █                                      2.8%

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

但如果算上 Peter 用 AI 辅助写的代码...

实际 AI 参与度可能 >>>>>>>>>>>>>>>>>>>>>>>>  ???%
```

---

## 9. 启示

### 对开发者

1. **AI 不只是写代码** —— 它可以做 PR review、CI/CD、日常维护
2. **Co-Author 标签是好习惯** —— 让 AI 贡献可追溯
3. **多 AI 协作是趋势** —— 不同 AI 擅长不同任务

### 对项目

1. **OpenClaw 是"自举"的** —— 用自己开发自己
2. **开源 + AI = 加速器** —— 社区 PR 被 AI 快速处理
3. **这可能是未来软件开发的样子**

---

## 10. 如何复现这些数据

```bash
# 总 commit 数
git log --all --oneline | wc -l

# 按作者统计
git log --all --format="%an <%ae>" | sort | uniq -c | sort -rn | head -20

# 查找 AI Co-Author
git log --all --format="%h %s%n%b" | grep -i "co-authored" | sort | uniq -c

# 查找 Bot 邮箱
git log --all --format="%ae" | grep -iE "bot|agent|clawd|openclaw" | sort | uniq -c
```

---

*最后更新：2026-02-06*

*数据来源：OpenClaw Git 仓库 commit 历史分析*
