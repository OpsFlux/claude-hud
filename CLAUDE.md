# CLAUDE.md

本文件用于在 Claude Code 处理该仓库时提供指引。

## 项目概览

Claude HUD 是一个 Claude Code 插件，用于显示实时的多行 statusline。它会展示上下文健康度、工具活动、代理状态以及 todo 进度。

## 构建命令

```bash
npm ci               # 安装依赖
npm run build        # 将 TypeScript 构建到 dist/

# 使用示例 stdin 数据测试
echo '{"model":{"display_name":"Opus"},"context_window":{"current_usage":{"input_tokens":45000},"context_window_size":200000}}' | node dist/index.js
```

## 架构

### 数据流

```
Claude Code → stdin JSON → parse → render lines → stdout → Claude Code displays
           ↘ transcript_path → parse JSONL → tools/agents/todos
```

**关键点**：Claude Code 大约每 300ms 会调用一次 statusline。每次调用都会：
1. 通过 stdin 接收 JSON（模型、上下文、tokens——原生且准确的数据）
2. 解析 transcript JSONL 文件，提取工具、代理与 todos
3. 将多行输出渲染到 stdout
4. 由 Claude Code 显示所有行

### 数据来源

**来自 stdin JSON 的原生数据**（准确，无需估算）：
- `model.display_name` - 当前模型
- `context_window.current_usage` - Token 计数
- `context_window.context_window_size` - 最大上下文窗口
- `transcript_path` - 会话 transcript 路径

**来自 transcript JSONL 的解析数据**：
- `tool_use` 块 → 工具名、输入、开始时间
- `tool_result` 块 → 完成情况、耗时
- 运行中的工具 = 没有匹配 `tool_result` 的 `tool_use`
- `TodoWrite` 调用 → todo 列表
- `Task` 调用 → 代理信息

**来自配置文件**：
- MCP 数量来自 `~/.claude/settings.json`（mcpServers）
- Hooks 数量来自 `~/.claude/settings.json`（hooks）
- Rules 数量来自各目录下的 CLAUDE.md 文件

### 目录结构

```
src/
├── index.ts           # 入口
├── stdin.ts           # 解析 Claude 的 JSON 输入
├── transcript.ts      # 解析 transcript JSONL
├── config-reader.ts   # 读取 MCP/rules 配置
├── types.ts           # TypeScript 接口类型
└── render/
    ├── index.ts          # 主渲染协调器
    ├── session-line.ts   # 第 1 行：model、context、rules、MCPs
    ├── tools-line.ts     # 第 2 行：工具活动
    ├── agents-line.ts    # 第 3 行：代理状态
    ├── todos-line.ts     # 第 4 行：todo 进度
    └── colors.ts         # ANSI 颜色辅助
```

### 输出格式

```
[Opus] ████████░░ 45% | 📋 3 rules | 🔌 5 MCPs | ⏱️ 12m
◐ Edit: auth.ts | ✓ Read ×3 | ✓ Grep ×2
◐ explore [haiku]: Finding auth code (2m 15s)
▸ Fix authentication bug (2/5)
```

各行会按条件显示：
- 第 1 行（session）：始终显示
- 第 2 行（tools）：有工具被使用时显示
- 第 3 行（agents）：仅在代理处于活跃状态时显示
- 第 4 行（todos）：仅在存在 todos 时显示

### 上下文阈值

| 阈值 | 颜色 | 行为 |
|-----------|-------|--------|
| <70% | 绿色 | 正常 |
| 70-85% | 黄色 | 警告 |
| >85% | 红色 | 显示 token 细分 |
| >95% | 红色 | 显示 ⚠️ COMPACT |

## 插件配置

插件清单（manifest）位于 `.claude-plugin/plugin.json`（仅包含元数据：name、description、version、author）。

**StatusLine 配置** 必须通过 `/claude-hud:setup` 写入用户的 `~/.claude/settings.json`。

setup 命令会添加一个自动更新的命令：在运行时找到本机已安装的最新版本。

注意：`statusLine` 不是 plugin.json 的有效字段。插件安装后必须在 settings.json 中进行配置。更新是自动的——无需重复执行 setup。

## 依赖

- **运行时**：Node.js 18+ 或 Bun
- **构建**：TypeScript 5，ES2022 target，NodeNext modules
