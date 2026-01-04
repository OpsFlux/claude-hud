# Claude HUD

一个 Claude Code 插件，用于显示当前正在发生的事情——上下文使用情况、活跃工具、正在运行的代理（agents）以及 todo 进度。始终在你的输入下方可见。

[![License](https://img.shields.io/github/license/jarrodwatts/claude-hud?v=2)](LICENSE)
[![Stars](https://img.shields.io/github/stars/jarrodwatts/claude-hud)](https://github.com/jarrodwatts/claude-hud/stargazers)

![Claude HUD in action](claude-hud-preview-5-2.png)

## 安装

在 Claude Code 实例中运行以下命令：

**第 1 步：添加 Marketplace**
```
/plugin marketplace add jarrodwatts/claude-hud
```

**第 2 步：安装插件**
```
/plugin install claude-hud
```

**第 3 步：配置状态栏（statusline）**
```
/claude-hud:setup
```

完成！HUD 会立刻出现——无需重启。

---

## Claude HUD 是什么？

Claude HUD 帮你更清晰地了解 Claude Code 会话中发生的事情。

| 你看到什么 | 为什么重要 |
|--------------|----------------|
| **上下文健康度** | 在为时已晚之前，精确知道上下文窗口还剩多少空间 |
| **工具活动** | 实时看到 Claude 正在读取、编辑、搜索哪些文件 |
| **代理追踪** | 查看哪些子代理正在运行，以及它们在做什么 |
| **Todo 进度** | 实时追踪任务完成情况 |

## 每一行显示什么

### 会话信息
```
[Opus 4.5] ████░░░░░░ 19% | 2 CLAUDE.md | 8 rules | 6 MCPs | 6 hooks | ⏱️ 1m
```
- **模型（Model）** — 当前使用的模型
- **上下文进度条（Context bar）** — 带颜色编码的可视化刻度（随着填满：绿 → 黄 → 红）
- **配置计数（Config counts）** — 已加载的 rules、MCPs 和 hooks 数量
- **持续时间（Duration）** — 会话已运行的时长

### 工具活动
```
✓ TaskOutput ×2 | ✓ mcp_context7 ×1 | ✓ Glob ×1 | ✓ Skill ×1
```
- **运行中的工具** 会显示一个转圈指示以及目标文件
- **已完成的工具** 会按类型聚合并统计次数

### 代理状态
```
✓ Explore: Explore home directory structure (5s)
✓ open-source-librarian: Research React hooks patterns (2s)
```
- 显示 **代理类型** 以及它正在处理的内容
- 显示每个代理的 **耗时**

### Todo 进度
```
✓ All todos complete (5/5)
```
- 显示 **当前任务** 或完成状态
- 显示 **进度计数**（已完成/总数）

---

## 工作原理

Claude HUD 使用 Claude Code 原生的 **statusline API** ——无需单独窗口、无需 tmux，在任何终端中都可用。

```
Claude Code → stdin JSON → claude-hud → stdout → displayed in your terminal
           ↘ transcript JSONL (tools, agents, todos)
```

**关键特性：**
- 直接使用 Claude Code 提供的原生 token 数据（非估算）
- 解析 transcript 以提取工具/代理活动
- 大约每 300ms 更新一次

---

## 环境要求

- Claude Code v1.0.80+
- Node.js 18+ 或 Bun

---

## 开发

```bash
git clone https://github.com/jarrodwatts/claude-hud
cd claude-hud
npm ci && npm run build
npm test
```

贡献指南请参见 [CONTRIBUTING.md](CONTRIBUTING.md)。

---

## 许可证

MIT ——参见 [LICENSE](LICENSE)。
