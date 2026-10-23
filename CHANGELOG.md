# 更新日志

本文件只记录 **fork 相对上游的改动**。上游更早的历史见
[上游仓库](https://github.com/MeteorNOX/DeepSeek-Balance-Whale-Widget) 的提交记录与 Releases。

## 0.5.2 — 2026-10-24

### 新增

- **「空闲时随机说话」有开关了**：右键挂件 →「自定义泡泡」窗口底部新增一行
  `☑ 空闲时随机说话　最短 [90] 秒　最长 [240] 秒`，随本窗口的「保存」一起落盘、立即生效，
  取消勾选＝不自动说话。此前这个功能只能在 `~/.dsh/.dshw-bubble.json` 的 `autoSay` 节里手改，
  界面上完全看不出来 —— 这是「装了 fork 却看不出差别」的主因。
  开关走的是泡泡配置保存的既有通道（宿主白名单里本来就有 `autoSay`），没有新增路由或接口。
  细节：[`docs/auto-say.md`](docs/auto-say.md)。

### 修复

- **版本号漂移（重要）**：`lib/index.js` 里写死的 `BUILD_VERSION = '0.4.2'` 从 0.5.0 起就没再跟着改，
  于是 0.5.0 / 0.5.1 装好后菜单底部仍显示 `0.4.2`、「关于 / 自检」面板同样如此，
  而更新检查 `hasUpdate: latest !== BUILD_VERSION` 永远为真 —— 每次启动都提示有新版。
  用户据此以为装到的是上游原版（本 fork 的痛点是"看不出差别"，这一条把它放大成了"看起来是旧包"）。
  - 现在版本号**只认同包 `package.json`**：运行时读取，读不到才退回 `BUILD_VERSION_FALLBACK`。
  - 另有两处 payload 里的 `version: '0.4.2'` 字面量一并改为 `version: BUILD_VERSION`。
- **新增门禁**：`tools/ci-audit.mjs` 第 ⑥ 项「检查版本号单一来源」——禁止 `const BUILD_VERSION = '…'`
  这类字面量、禁止 payload 里写死语义化版本、要求 `BUILD_VERSION_FALLBACK` 必须等于
  `package.json` 的 `version`。`tools/ci-audit-selftest.mjs` 同时补了 3 个用例（漂移必红 / 写死必红 / 干净必绿）。

### 说明

- 包内容变化：`assets/whale-widget.js` 新增自动说话开关与间隔输入（5 处 `// [patch:auto-say]` 标记）；
  其余行为与 0.5.1 一致。
- `ttlMs` / `idleMs` 这两项没有做进界面，需要时仍可手改 `~/.dsh/.dshw-bubble.json`；
  界面保存时会原样保留它们的值（不会被覆盖回默认）。

## 0.5.1 — 2026-10-23

发布链路改动，**包内容与 `0.5.0` 完全相同**（`assets/whale-widget.js`、`lib/index.js` 逐字节未变）。

### 修复

- 发版改为纯 **npm Trusted Publishing (OIDC)**：GitHub Actions 用 `id-token` 换一次性凭证，
  仓库里不需要任何 token。
- 工作流里的 `NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}` 是个坑：secret 未配置时它展开成空串，
  npm 会拿空 token 去请求而不是回退到 OIDC。已移除。
- `--provenance=false` 会把 OIDC 发布时**默认自动生成**的 provenance 证明关掉，已移除。
  从本版起，CI 发布的包都会带 provenance，npm 包页面显示「Built and signed on GitHub Actions」。
- `tools/ci-audit.mjs` 解析 `cordis.patch.yml` 里的 `name` 时没有剥掉 YAML 引号：scoped 包名
  必须写成 `name: '@scope/name'`（`@` 是 YAML 保留字符），于是发版门禁第 [3] 项误报
  「声明名与 package.json 不一致」，本地用 `--no-pack` 又测不到。已改为剥引号后比较。

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
