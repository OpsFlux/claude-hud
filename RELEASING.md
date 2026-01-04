# 发布流程

本项目以 Claude Code 插件形式发布。发布时应包含已编译的 `dist/` 输出。

## 发布检查清单

1) 更新版本号：
   - `package.json`
   - `.claude-plugin/plugin.json`
   - `CHANGELOG.md`
2) 构建：
   ```bash
   npm ci
   npm run build
   npm test
   npm run test:coverage
   ```
3) 验证插件入口：
   - `.claude-plugin/plugin.json` 指向 `dist/index.js`
4) 提交并打 tag：
   - `git tag vX.Y.Z`
5) 发布：
   - 推送 tag
   - 基于 `CHANGELOG.md` 的内容创建 GitHub Release 并填写说明
