# 更新日志

Claude HUD 的所有重要变更都会记录在此文件中。

## [2.0.0] - 2025-01-02

### 变更
- 从分屏 TUI 完整重写为内联 statusline
- 全新的 statusline 渲染器，支持多行输出
- 基于 transcript 的 tool/agent/todo 解析
- 从 stdin JSON 读取原生上下文使用数据

### 移除
- 基于 Hook 的捕获流程
- 分屏 UI 及相关组件
