# 测试策略

本项目规模较小、运行于终端环境，且大多数行为是确定性的。本测试策略侧重于快速、可靠的检查，用于验证核心行为，并为 PR 提供安全的合并门槛。

## 目标

- 以确定性的方式验证核心逻辑（解析、聚合、格式化）。
- 在不依赖人工审查的情况下捕获 HUD 输出的回归。
- 保持测试执行快速（<5s），方便贡献者频繁运行。

## 测试分层

1) 单元测试（快、确定性强）
- 纯辅助函数：`getContextPercent`、`getModelName`、token/耗时格式化等。
- 渲染辅助：字符串拼接与截断行为。
- Transcript 解析：tool/agent/todo 聚合与会话开始时间检测。

2) 集成测试（CLI 行为）
- 使用示例 stdin JSON 与 fixture transcript 运行 CLI。
- 验证渲染输出包含预期标记（模型、百分比、工具名等）。
- 断言对轻微格式变化保持鲁棒（避免严格整行匹配）。

3) Golden-output 测试（近期规划）
- 对已知 fixtures 对比完整输出快照，以捕获细微 UI 回归。
- 仅在输出变更是“有意为之”时更新快照。

## 优先测试内容

- Transcript 解析（tool use/result 映射、todo 提取）。
- 上下文百分比计算（包含 cache tokens）。
- 截断与聚合逻辑（tools/todos/agents 的展示规则）。
- 异常或不完整输入（错误 JSON 行、缺失字段）。

## Fixtures

- 共享测试数据放在 `tests/fixtures/` 下。
- 使用小型 JSONL 文件，每个文件聚焦一个行为（例如：基础工具流程、代理生命周期、todo 更新）。

## 本地运行测试

```bash
npm test
```

该命令会先运行 `npm run build`，然后执行 Node 内置的测试运行器。

生成覆盖率：

```bash
npm run test:coverage
```

更新快照：

```bash
npm run test:update-snapshots
```

## CI 门禁（推荐）

- `npm ci`
- `npm run build`
- `npm test`

仓库自带的 GitHub Actions 工作流会在 Node 18 和 20 上运行 `npm run test:coverage`。

建议将上述步骤设为 PR 检查的必需项，以确保新变更不会回归既有行为。

## 贡献期望

- 行为变更需新增或更新测试。
- 新增辅助函数优先写单元测试；用户可见输出的变更优先写集成测试。
- 保持测试确定性，避免依赖时间的断言（除非有可控手段）。
