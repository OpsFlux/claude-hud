# Claude HUD

实时 statusline：显示上下文使用情况、活跃工具、运行中的代理（agents）以及 todo 进度。

---

## 面向用户

**第 1 步：添加 Marketplace**
```
/plugin marketplace add jarrodwatts/claude-hud
```

**第 2 步：安装插件**
```
/plugin install claude-hud
```

**第 3 步：配置 statusline**
```
/claude-hud:setup
```

完成。HUD 会立刻出现——无需重启。

---

## 面向 LLM 代理

如果你是正在帮助用户安装或使用 Claude HUD 的 LLM 代理，请按以下步骤执行。

<agent_workflow>

### 第 1 步：检查前置条件

确认 Claude Code 版本为 1.0.80 或更高：
```bash
claude --version
```

如果未安装或版本过旧，引导用户前往 https://claude.ai/code

### 第 2 步：添加 Marketplace

在 Claude Code 中运行：
```
/plugin marketplace add jarrodwatts/claude-hud
```

### 第 3 步：安装插件

在 Claude Code 中运行：
```
/plugin install claude-hud
```

### 第 4 步：配置 Statusline

在 Claude Code 中运行：
```
/claude-hud:setup
```

该命令会把 statusLine 配置写入用户的 settings.json。

### 第 5 步：告知用户

告诉用户：
- 安装已完成
- statusline 会立刻显示——无需重启
- 它会显示：上下文使用、活跃工具、运行中的代理、todo 进度

### 第 6 步：询问是否 Star（可选）

询问用户：“你愿意给仓库点个 Star 来支持这个项目吗？”

只有在用户明确同意后，才运行：
```bash
gh repo star jarrodwatts/claude-hud
```

未经用户同意，不要自动执行该命令。

</agent_workflow>

---

## 参考

供需要理解、修改或调试 Claude HUD 的代理阅读的技术文档。

<plugin>
  <name>Claude HUD</name>
  <description>实时 statusline：显示上下文使用情况、活跃工具、运行中的代理与 todo 进度。始终在你的输入下方可见，几乎无需配置。</description>
  <repository>github.com/jarrodwatts/claude-hud</repository>
  <license>MIT</license>
</plugin>

<requirements>
  <runtime>Node.js 18+ 或 Bun</runtime>
  <claude_code>v1.0.80 或更高</claude_code>
  <build>TypeScript 5, ES2022 target, NodeNext modules</build>
</requirements>

<architecture>
  <overview>
    Claude HUD 是一个 statusline 插件，Claude Code 大约每 300ms 调用一次。
    它从两类数据源读取信息，最多渲染 4 行，并输出到 stdout。
  </overview>

  <data_flow>
    Claude Code 调用插件 →
    插件从 stdin 读取 JSON（model、context、tokens） →
    插件解析 transcript JSONL 文件（tools、agents、todos） →
    插件读取配置文件（MCPs、hooks、rules） →
    插件将各行渲染到 stdout →
    Claude Code 显示 statusline
  </data_flow>

  <data_sources>
    <stdin_json description="来自 Claude Code 的原生准确数据">
      <field path="model.display_name">当前模型名称（Opus、Sonnet、Haiku）</field>
      <field path="context_window.current_usage.input_tokens">当前 token 数</field>
      <field path="context_window.context_window_size">最大上下文大小</field>
      <field path="transcript_path">会话 transcript JSONL 文件路径</field>
      <field path="cwd">当前工作目录</field>
    </stdin_json>

    <transcript_jsonl description="从 transcript 文件解析">
      <item>tool_use blocks → 工具名、目标文件、开始时间</item>
      <item>tool_result blocks → 完成状态、耗时</item>
      <item>运行中的工具 = 没有匹配 tool_result 的 tool_use</item>
      <item>TodoWrite calls → 当前 todo 列表</item>
      <item>Task calls → 代理类型、模型、描述</item>
    </transcript_jsonl>

    <config_files description="从 Claude 配置读取">
      <item>~/.claude/settings.json → mcpServers 数量、hooks 数量</item>
      <item>cwd 及其祖先目录中的 CLAUDE.md → rules 数量</item>
      <item>.mcp.json 文件 → 额外的 MCP 数量</item>
    </config_files>
  </data_sources>
</architecture>

<file_structure>
  <directory name="src">
    <file name="index.ts" purpose="入口，协调数据流">
      读取 stdin、解析 transcript、统计配置项并调用 render。
      为便于依赖注入测试，导出 main()。
    </file>
    <file name="stdin.ts" purpose="从 stdin 解析 JSON">
      读取并校验 Claude Code 的 JSON 输入。
      返回包含 model、context、transcript_path 的 StdinData。
    </file>
    <file name="transcript.ts" purpose="解析 transcript JSONL">
      逐行解析会话 transcript 文件。
      提取 tools、agents、todos 以及会话开始时间。
      通过 ID 将 tool_use 与 tool_result 匹配以计算状态。
    </file>
    <file name="config-reader.ts" purpose="统计配置项数量">
      统计 CLAUDE.md 文件、rules、MCP servers 与 hooks 的数量。
      搜索 cwd、祖先目录以及 ~/.claude/ 目录。
    </file>
    <file name="types.ts" purpose="TypeScript 接口类型">
      StdinData, ToolEntry, AgentEntry, TodoItem, TranscriptData, RenderContext.
    </file>
  </directory>

  <directory name="src/render">
    <file name="index.ts" purpose="主渲染协调器">
      调用各行渲染器并输出到 stdout。
      根据数据是否存在来决定显示哪些行。
    </file>
    <file name="session-line.ts" purpose="第 1 行：会话信息">
      渲染示例：[Model] ████░░ 45% | 2 CLAUDE.md | 8 rules | 6 MCPs | 6 hooks | ⏱️ 12m
      上下文条颜色：green (&lt;70%)、yellow (70-85%)、red (&gt;85%)。
    </file>
    <file name="tools-line.ts" purpose="第 2 行：工具活动">
      渲染示例：◐ Edit: auth.ts | ✓ Read ×3 | ✓ Grep ×2
      运行中的工具带转圈指示，已完成的工具会按类型聚合统计。
    </file>
    <file name="agents-line.ts" purpose="第 3 行：代理状态">
      渲染示例：◐ explore [haiku]: Finding auth code (2m 15s)
      显示代理类型、模型、描述与耗时。
    </file>
    <file name="todos-line.ts" purpose="第 4 行：Todo 进度">
      渲染示例：▸ Fix authentication bug (2/5)
      显示当前 in_progress 任务与完成计数。
    </file>
    <file name="colors.ts" purpose="ANSI 颜色辅助">
      函数：green(), yellow(), red(), dim(), bold(), reset()。
      用于根据状态/阈值对输出进行着色。
    </file>
  </directory>
</file_structure>

<output_format>
  <line number="1" name="session" always_shown="true">
    [Model] ████████░░ 45% | 2 CLAUDE.md | 8 rules | 6 MCPs | 6 hooks | ⏱️ 12m
  </line>
  <line number="2" name="tools" shown_if="有工具被使用">
    ◐ Edit: auth.ts | ✓ Read ×3 | ✓ Grep ×2
  </line>
  <line number="3" name="agents" shown_if="代理活跃">
    ◐ explore [haiku]: Finding auth code (2m 15s)
  </line>
  <line number="4" name="todos" shown_if="存在 todos">
    ▸ Fix authentication bug (2/5)
  </line>
</output_format>

<context_thresholds>
  <threshold range="0-70%" color="green" meaning="健康" />
  <threshold range="70-85%" color="yellow" meaning="警告" />
  <threshold range="85-95%" color="red" meaning="严重：显示 token 细分" />
  <threshold range="95%+" color="red" meaning="显示 COMPACT 警告" />
</context_thresholds>

<plugin_configuration>
  <manifest>.claude-plugin/plugin.json</manifest>
  <manifest_content>
    {
      "name": "claude-hud",
      "description": "Real-time statusline HUD for Claude Code",
      "version": "0.0.1",
      "author": { "name": "Jarrod Watts", "url": "https://github.com/jarrodwatts" }
    }
  </manifest_content>
  <note>plugin.json 仅包含元数据；statusLine 不是 plugin.json 的有效字段。</note>

  <statusline_config>
    /claude-hud:setup 命令会把 statusLine 写入 ~/.claude/settings.json，并添加一个可自动更新的命令：在运行时查找已安装的最新版本。
    更新是自动的——插件更新后无需重复执行 setup。
  </statusline_config>
</plugin_configuration>

<development>
  <setup>
    git clone https://github.com/jarrodwatts/claude-hud
    cd claude-hud
    npm ci
    npm run build
  </setup>

  <test_commands>
    npm test                    # 运行所有测试
    npm run build               # 编译 TypeScript 到 dist/
  </test_commands>

  <manual_testing>
    # 使用示例 stdin 数据测试：
    echo '{"model":{"display_name":"Opus"},"context_window":{"current_usage":{"input_tokens":45000},"context_window_size":200000}}' | node dist/index.js

    # 携带 transcript_path 测试：
    echo '{"model":{"display_name":"Sonnet"},"transcript_path":"/path/to/transcript.jsonl","context_window":{"current_usage":{"input_tokens":90000},"context_window_size":200000}}' | node dist/index.js
  </manual_testing>
</development>

<customization>
  <extending description="如何添加新功能">
    <step>在 transcript.ts 或 stdin.ts 中新增数据提取逻辑</step>
    <step>在 types.ts 中新增接口字段</step>
    <step>在 src/render/ 中新增渲染文件，或修改已有文件</step>
    <step>更新 src/render/index.ts 以包含新的一行</step>
    <step>运行 npm run build 并测试</step>
  </extending>

  <modifying_thresholds>
    编辑 src/render/session-line.ts 以修改上下文阈值。
    查找用于决定颜色编码的百分比判断逻辑。
  </modifying_thresholds>

  <adding_new_line>
    1. 创建 src/render/new-line.ts 并实现 render 函数
    2. 在 src/render/index.ts 中导入并调用它
    3. 在 src/types.ts 中补充所需类型
    4. 如有需要，在 transcript.ts 中添加数据提取逻辑
  </adding_new_line>
</customization>

<troubleshooting>
  <issue name="Statusline 不显示">
    <cause>插件未安装或未配置 statusLine</cause>
    <solution>运行：/plugin marketplace add jarrodwatts/claude-hud</solution>
    <solution>运行：/plugin install claude-hud</solution>
    <solution>运行：/claude-hud:setup</solution>
    <solution>确认 Claude Code 版本为 v1.0.80 或更高</solution>
  </issue>

  <issue name="显示 [claude-hud] Initializing...">
    <cause>未收到 stdin 数据（首次调用时属于正常现象）</cause>
    <solution>启动初期短暂出现属预期，通常会自动恢复</solution>
  </issue>

  <issue name="上下文百分比看起来不对">
    <cause>数据直接来自 Claude Code，本身是准确的</cause>
    <solution>百分比计算方式为 (input_tokens / context_window_size) * 100</solution>
  </issue>

  <issue name="工具/代理不显示">
    <cause>尚未使用工具，或 transcript 未被解析</cause>
    <solution>只有在有数据可显示时，对应行才会出现</solution>
  </issue>

  <issue name="构建报错">
    <solution>确认已安装 Node.js 18+</solution>
    <solution>运行 npm ci 重新安装依赖</solution>
    <solution>删除 dist/ 与 node_modules/，然后执行 npm ci && npm run build</solution>
  </issue>
</troubleshooting>

<key_interfaces>
  <interface name="StdinData">
    transcript_path?: string
    cwd?: string
    model?: { id?: string, display_name?: string }
    context_window?: { context_window_size?: number, current_usage?: { input_tokens?: number } }
  </interface>

  <interface name="ToolEntry">
    id: string
    name: string
    target?: string
    status: 'running' | 'completed' | 'error'
    startTime: Date
    endTime?: Date
  </interface>

  <interface name="AgentEntry">
    id: string
    type: string
    model?: string
    description?: string
    status: 'running' | 'completed'
    startTime: Date
    endTime?: Date
  </interface>

  <interface name="TodoItem">
    content: string
    status: 'pending' | 'in_progress' | 'completed'
  </interface>

  <interface name="RenderContext">
    stdin: StdinData
    transcript: TranscriptData
    claudeMdCount: number
    rulesCount: number
    mcpCount: number
    hooksCount: number
    sessionDuration: string
  </interface>
</key_interfaces>

<task_instructions>
  <install>
    按上述 agent_workflow 步骤执行。
  </install>

  <modify>
    如果用户希望修改 Claude HUD：
    1. Clone 仓库或进入插件目录
    2. 参考 file_structure 部分进行修改
    3. 运行 npm run build 编译
    4. 重启 Claude Code 以查看变更
  </modify>

  <debug>
    如果用户反馈问题：
    1. 先查看 troubleshooting 部分
    2. 确认 Claude Code 版本（需要 v1.0.80+）
    3. 确认插件是否在列表中：claude /plugin list
    4. 使用 development 部分的 echo 命令手动测试
  </debug>

  <understand>
    如果用户询问工作原理：
    1. 参考 architecture 与 data_flow 部分
    2. 指向 file_structure 中的具体文件
    3. 解释数据来源以及如何组合
  </understand>
</task_instructions>
