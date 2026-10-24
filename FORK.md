# 这个仓库是什么

[`MeteorNOX/DeepSeek-Balance-Whale-Widget`](https://github.com/MeteorNOX/DeepSeek-Balance-Whale-Widget)
的**增强 fork**，基线是上游 tag `v0.4.2`（2026-10-11，MIT，上游作者 MeteorNOX）。

不是重写，也不是上游的另一个分支。除了下面列出的三处功能改动、一处宿主放行和新增的文档之外，
其余文件（`lib/accounting.mjs`、`tools/`、`locale`/素材、README、`PROVENANCE.md`、CI）与上游
`v0.4.2` 完全相同。想核对可以按「怎么核对」一节跑一遍 diff。

想要原版就用上游仓库；这里的改动都写在下面，全部可以单独摘出去。

## 相对上游新增

### 1. 空闲自动说话（`autoSay`）

小鲸鱼在**没人动鼠标、没有气泡正在显示**的时候，自己冒一句随机语句，到时间收起来。
句子池就是「随机语句」模块里的那些句子，用户改过的话按用户改的来。

**开关在界面上**：右键挂件 →「自定义泡泡」窗口底部 `☑ 空闲时随机说话　最短 [60] 秒　最长 [120] 秒`，
点该窗口的「保存」立即生效（不需要重启或刷新）。`assets/whale-widget.js` 里以
`// [patch:auto-say]` 标记的 5 处就是这个开关；它复用了泡泡配置保存的既有通道。

配置也可以直接写在 `.dshw-bubble.json` 的 `autoSay` 节（`.dshw-bubble.json` 与 `.dshw-usage.json`
在同一目录，通常是 `%USERPROFILE%\.dsh\`）：

```json
"autoSay": { "on": true, "minMs": 60000, "maxMs": 120000, "ttlMs": 8000, "idleMs": 0 }
```

| 字段 | 含义 | 允许范围（超出回落到默认） | 默认 |
|---|---|---|---|
| `on` | 开关，`false` 关闭（不删配置即可暂停） | `true` / `false` | `true` |
| `minMs` / `maxMs` | 两次自动说话的间隔区间（毫秒），每次在区间内随机 | 5000 – 86400000 | 60000 / 120000 |
| `ttlMs` | 一句停留多久（毫秒） | 1000 – 60000 | 8000 |
| `idleMs` | 用户多久没动鼠标才算「空闲」（毫秒），0 = 不等 | 0 – 86400000 | 0 |

细节见 [`docs/auto-say.md`](docs/auto-say.md)。

### 2. 随机语句「洗牌袋」

上游的随机语句是**每次独立加权随机**：权重只影响长期概率，所以短时间里同一句可能连着出现好几次，
权重 10 的句子也不保证在一轮里真的出现 10 次。

这里换成一个**洗牌袋**：把每条语句按权重复制成票（`w` 份），整袋 Fisher-Yates 洗牌后顺序取出，
取空再装新的一袋。于是

- 一轮之内**每条语句都出现恰好 `w` 次**，不会漏，也不会超出；
- 同一句**不会紧挨着出现两次**（袋内去相邻重复 + 袋首避开上一袋的末尾 12 个）；
- 权重总和大到离谱时（> 20000 票，比如用户把权重写成天文数字）自动退回「加权随机 + 避开上一泡」，
  不会为了装袋子吃掉内存。

### 3. 宿主放行 `autoSay`

宿主的「保存泡泡序列」路由是按白名单落盘的。上游不认识 `autoSay`，所以手写进
`.dshw-bubble.json` 的配置在用户点一次「保存」之后就会被清掉 —— 功能看起来像自己坏了。
本 fork 在 `lib/index.js` 里显式放行这个键并做净化（范围同上表），前端保存时把它原样带回去。

### 4. 出厂默认泡泡配置

`BUBBLE_DEFAULT_ITEMS`（`assets/whale-widget.js`）不再是上游那套，而是一份真实在用配置的快照：

- 首泡：今日已用 / 余额 / 峰谷时段 / 下一时段倒计时；
- 第二泡是并列二选一：左侧**203 条**随机语句（全部 `w: 1`），右侧 `randimg`（`petpet` / `money1` 等权）。

影响范围只有三类场景：全新安装、删掉 `~/.dsh/.dshw-bubble.json`、点「恢复默认」。
已经有配置的用户**完全不受影响** —— 读取顺序仍然是「用户配置优先，没有才用出厂池」。

顺便说清一件事：这 203 条里**没有**上游那三档权重（10 / 3 / 1）。权重是编辑器保存时写平的（全部 `w: 1`），
所以出厂池是均匀的；想要强调某几句，在「自定义泡泡」里改那几条的 `w` 即可。
## 没改的东西

- 余额取数、今日已用记账、峰谷定价、每轮消耗、音频、拖拽吸附、资源管理 —— 一行没动。
- 所有后台路由、z 层级登记、动态 HTML 写入白名单 —— 没动，`tools/ci-audit.mjs`、
  `tools/z-layer-audit.mjs`、`tools/check-dead-settings.mjs`、`tools/ci-audit-selftest.mjs`
  全部照旧通过（CI 见 `.github/workflows/ci.yml`）。
- 素材（`assets/**`、`icon.png`）**没有**重新编码、也没有新增；它们的授权与上游
  [`PROVENANCE.md`](PROVENANCE.md) 一致：**不适用 MIT**，随插件 as-is 分发，原作者可随时要求下线。

## 怎么核对

```sh
git remote add upstream https://github.com/MeteorNOX/DeepSeek-Balance-Whale-Widget.git
git fetch upstream --tags
git diff upstream/v0.4.2 --stat
```

应当只看到 `assets/whale-widget.js`、`lib/index.js`，以及本文件、`CHANGELOG.md`、
`docs/auto-say.md`、`README.md` 的横幅这几处。

其中 `assets/whale-widget.js` 的三处改动都可以用标记定位（默认池是数据，不打标记）：

```sh
grep -n 'patch:auto-say\|dshwPickBag' assets/whale-widget.js
```

## 怎么和上游同步

```sh
git fetch upstream --tags
git merge upstream/v0.4.3          # 或上游的新 tag
```

冲突只会出现在 `assets/whale-widget.js`（如果上游也改到 `bubblePickLine()` 或
`loadBubbleCfg()` 附近）和 `lib/index.js` 的泡泡保存路由。改完照常跑 CI：

```sh
node tools/z-layer-audit.mjs assets/whale-widget.js
node tools/ci-audit.mjs --no-pack
node tools/ci-audit-selftest.mjs
node tools/check-dead-settings.mjs
```

## 包名与仓库地址

本 fork 已挂在 `@tuweizhi` 名下：

- 包名 `@tuweizhi/dsh-whale-widget`（上游是 `dsh-whale-widget`，两者会并存冲突，装之前先移除旧的）
- 仓库 `https://github.com/TuWeiZhi/dsh-whale-widget`
- 安装见 README 顶部的「安装这个 fork」一节

要换成别的 scope 时，**别只改 `package.json`**——包名散在下面这些地方：

```sh
grep -rn 'tuweizhi' package.json cordis.patch.yml README.md lib/index.js .github/workflows/publish.yml
```

| 位置 | 改什么 |
|---|---|
| `package.json` | `name`、`repository.url`、`bugs.url`、`homepage` |
| `cordis.patch.yml` | `name:`（bundle 里实际加载的包名） |
| `lib/index.js` | ① 插件注册的 `name:`；② 更新检查器的四个 `UPDATE_SOURCES` URL、`npm view <name> version`、`user-agent` |
| `.github/workflows/publish.yml` | 「版本是否已发布」那步的 registry URL |
| `README.md` | 顶部「安装这个 fork」里的三条命令 |

⚠️ **scoped 包名在 registry API 里要写 `%2f`**：`registry.npmmirror.com/@tuweizhi%2fdsh-whale-widget/latest`、
`registry.npmjs.org/@tuweizhi%2fdsh-whale-widget/<version>`。jsdelivr / unpkg 反而**不编码**
（`cdn.jsdelivr.net/npm/@tuweizhi/dsh-whale-widget@latest/package.json`）。写错的后果很轻但不明显：
更新检查永远 404，界面上只是一句「检查更新失败」。这些地址本仓库已经按规则改好了，
换 scope 时照着改即可。

## 许可

代码 MIT，见 [`LICENSE`](LICENSE)（保留上游版权行）。素材的额外限制见
[`PROVENANCE.md`](PROVENANCE.md)。本 fork 的改动同样以 MIT 提供。
