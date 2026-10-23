# 更新日志

本文件只记录 **fork 相对上游的改动**。上游更早的历史见
[上游仓库](https://github.com/MeteorNOX/DeepSeek-Balance-Whale-Widget) 的提交记录与 Releases。

## 0.5.1 — 2026-10-23

发布链路改动，**包内容与 `0.5.0` 完全相同**（`assets/whale-widget.js`、`lib/index.js` 逐字节未变）。

### 修复

- 发版改为纯 **npm Trusted Publishing (OIDC)**：GitHub Actions 用 `id-token` 换一次性凭证，
  仓库里不需要任何 token。
- 工作流里的 `NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}` 是个坑：secret 未配置时它展开成空串，
  npm 会拿空 token 去请求而不是回退到 OIDC。已移除。
- `--provenance=false` 会把 OIDC 发布时**默认自动生成**的 provenance 证明关掉，已移除。
  从本版起，CI 发布的包都会带 provenance，npm 包页面显示「Built and signed on GitHub Actions」。

## 0.5.0 — 2026-10-23

基线：上游 `v0.4.2`（2026-10-11）。

### 新增

- **空闲自动说话**（`patch:auto-say`）：空闲时小鲸鱼自己冒一句随机语句。
  配置在 `.dshw-bubble.json` 的 `autoSay` 节（`on` / `minMs` / `maxMs` / `ttlMs` / `idleMs`），
  每次调度前重新读取，改配置后最多一个周期生效，无需刷新。详见 [`docs/auto-say.md`](docs/auto-say.md)。
- **随机语句「洗牌袋」**：`bubblePickLine()` 由「每次独立加权随机」改为按权重装袋 + 洗牌后顺序取出，
  一轮内每条语句按权重出现恰好 `w` 次、相邻不重复、袋首避开上一袋末尾 12 个；
  权重总和 > 20000 时自动退回「加权随机 + 避开上一泡」，避免为装袋吃掉内存。
- 宿主放行 `autoSay`：`lib/index.js` 的泡泡保存路由原本按白名单落盘，会把这个键整份丢掉，
  导致手写配置在用户点一次「保存」后消失。现在显式放行并净化（范围与前端默认值一致），
  前端保存时原样回传。
- 文档：[`FORK.md`](FORK.md)（这个仓库是什么、怎么核对、怎么同步上游）、
  [`docs/auto-say.md`](docs/auto-say.md)（自动说话配置）、本文件。

### 未改动

余额取数、今日已用记账、峰谷定价、每轮消耗、音频、拖拽吸附、资源管理、后台路由、
z 层级登记、动态 HTML 写入白名单 —— 全部与上游 `v0.4.2` 一致。

### 验证

```sh
node tools/z-layer-audit.mjs assets/whale-widget.js
node tools/ci-audit.mjs --no-pack
node tools/ci-audit-selftest.mjs
node tools/check-dead-settings.mjs
```

四条全绿（与上游 CI 的检查项一致，未放宽任何一条）。
