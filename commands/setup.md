---
description: 将 claude-hud 配置为你的 statusline
allowed-tools: Bash, Read, Edit, AskUserQuestion
---

将以下 statusLine 配置添加到 `~/.claude/settings.json`：

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash -c 'node \"$(ls -td ~/.claude/plugins/cache/claude-hud/claude-hud/*/ 2>/dev/null | head -1)dist/index.js\"'"
  }
}
```

该命令会自动找到并运行本机已安装的最新版本。

请与现有 settings 合并，不要覆盖其他字段。

HUD 会立刻出现——无需重启。更新是自动的。

配置成功后，询问用户是否愿意给仓库点 ⭐ Star 来支持该项目。

如果用户同意，执行：`gh api -X PUT /user/starred/jarrodwatts/claude-hud`

只有在用户明确表示同意时才执行 Star 命令。
