# 自动说话（autoSay）

空闲时让右下角的小鲸鱼自己冒一句随机语句。这是本 fork 新增的功能，上游没有。

## 配置

配置写在 `.dshw-bubble.json` 的 `autoSay` 节。这个文件就在 `.dshw-usage.json` 旁边：

- 桌面端：`%USERPROFILE%\.dsh\.dshw-bubble.json`
- 网页端：`%USERPROFILE%\.dsh\profiles\web\.dshw-bubble.json`

改完**不用重启、不用刷新**：下一次调度就会读到新值。最坏情况下等一个 `maxMs` 周期。

```json
{
  "v": 1,
  "items": [ ... ],
  "lib": [ ... ],
  "tapAdvance": false,
  "autoSay": { "on": true, "minMs": 90000, "maxMs": 240000, "ttlMs": 6500, "idleMs": 20000 }
}
```

只关心 `autoSay` 这一节，别动其它键（`items` / `lib` 是泡泡序列本身，见 README）。

| 字段 | 含义 | 范围 | 默认 |
|---|---|---|---|
| `on` | 总开关 | 布尔 | `true` |
| `minMs` | 间隔下限（毫秒） | 5000 – 86400000 | `90000`（1.5 分钟） |
| `maxMs` | 间隔上限（毫秒），每次在 `[minMs, maxMs)` 里随机 | ≥ `minMs`，≤ 86400000 | `240000`（4 分钟） |
| `ttlMs` | 一句话停留多久（毫秒） | 1000 – 60000 | `6500` |
| `idleMs` | 多久没动鼠标才算空闲（毫秒），`0` = 不等 | 0 – 86400000 | `20000` |

越界的值会被宿主夹到范围内；`maxMs < minMs` 时按 `minMs` 处理。整节缺失、或 JSON 里没有这个键，
就是出厂默认值（开启，1.5–4 分钟一句）。

## 什么时候不说话

只要满足任意一条就跳过本轮，10 秒后再看：

- 已经有气泡在显示（`bubbleShown`）；
- 页面在后台（`document.hidden`）；
- 有提问/等待气泡正在打字（`waitShown`）；
- 距离上次鼠标移动/点击/滚轮/按键还不到 `idleMs`。

关掉（`"on": false`）之后它每 60 秒看一眼配置，所以随时改回 `true` 都能自己恢复，不用刷新页面。

## 句子从哪来

1. 用户配置的「随机语句」模块（`.dshw-bubble.json` 里 `items[1].options[0].item.modules` 中第一个
   `type: "random"` 的模块）的 `lines`；
2. 没有就用出厂默认的随机语句池。

取句走的是「洗牌袋」（见 [`FORK.md`](../FORK.md)）：一轮里每条按权重出现，不会连着重复。
袋子按「同一个语句数组」记账，所以同一份池子里的取句是同一轮轮转的一部分。

## 调试

```js
window.__dshwAutoSay.next()      // 立刻说一句（不等间隔）
window.__dshwAutoSay.config()    // 当前生效的配置
window.__dshwAutoSayDbg          // 上一次自动说话的时间与内容
```

## 关掉它

三种方式，任选：

1. `.dshw-bubble.json` 里 `"on": false`（保留配置，随时能开回来）；
2. 删掉 `autoSay` 这一节（回到默认值，也就是开启）；
3. 用上游原版 —— 这是本 fork 新增的功能，上游没有这段代码。
