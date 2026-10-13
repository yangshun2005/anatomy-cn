# 验证

在别人之前，先以一位严苛审稿人的眼光审视图形。渲染成本很低，而猜测代价高昂。

## 截图
`scripts/capture.mjs` 会启动自己的无头 Chrome（如果找不到，请设置 `CHROME_PATH`），加载文件或 URL，默认裁剪到 `.iso-plate`。它还会打印读数、任何控制台错误或异常，以及页面是否横向滚动。

```sh
node scripts/capture.mjs page.html --out shots/desk.png                         # 桌面端 1440，dpr 2
node scripts/capture.mjs page.html --out shots/phone.png --width 390            # 手机端
node scripts/capture.mjs page.html --out shots/hover.png --hover 0.7,0.5        # 指针位于横向 70% 处
node scripts/capture.mjs page.html --out shots/tour.png --times 1200,2400,3600  # 运动：随时间额外截图
node scripts/capture.mjs page.html --out shots/keys.png --key ArrowRight,ArrowRight
node scripts/capture.mjs page.html --out shots/joint.png --zoom 200,80,220,170  # 4 倍特写，以卡片像素为单位
node scripts/capture.mjs page.html --out shots/still.png --reduced              # 减弱动效
node scripts/capture.mjs http://localhost:3000/docs --select "figure:nth-of-type(3) .iso-plate"   # 开发服务器页面上的单个图形
```
`--select` 接受任意 CSS 选择器；默认裁剪到第一个 `.iso-plate`。每个 `--times` 延迟都从第一次截图开始计时，因此 `1200,2400,3600` 大约在第一次截图后 1.2、2.4 和 3.6 秒时截图（一次截图需要 60–200 毫秒，因此比这更近的延迟会延后执行），保存为 `name-<ms>.png`。`--key` 发送与 `drive.mjs` 相同的键码（`7` 为 `Digit7`，`.` 为 `Period`；逗号写作 `Comma`，因为列表以逗号分隔）。其他选项有 `--height`（1000）、`--scale`（2）、`--wait`（第一次截图前等待 2500 毫秒）、`--zoom-scale`（4）和 `--theme dark|light`。读取你生成的每一张 PNG：打开它并仔细观察。

两个脚本启动 Chrome 时都开启 WebGL（`--enable-unsafe-swiftshader --ignore-gpu-blocklist`），因此着色器也会出现在 `capture.mjs` 的静态图中。对于任何需要计时或放慢的内容，请使用 `drive.mjs`。

## 驱动
`scripts/drive.mjs` 在开启 WebGL 的无头 Chrome 中运行脚本化会话。用于序列：悬停、点击、输入、等待某个阶段。也用于捕捉突变的放慢接触表，以及无 WebGL 回退。它在每次截图后打印读数，然后打印任何控制台错误和异常。

```sh
node scripts/drive.mjs page.html '[["wait",1500],["move",".rt-stage",0.7,0.4],["wait",600],["shot","shots/hover.png"]]'
node scripts/drive.mjs page.html --actions steps.json --height 1300
node scripts/drive.mjs page.html --actions steps.json --width 390
node scripts/drive.mjs page-light.html --actions steps.json --reduced
node scripts/drive.mjs page.html --actions steps.json --no-webgl
node scripts/drive.mjs page-light.html --actions steps.json --preload slow.js
```
选项有 `--width`（1440）、`--height`（1100）、`--scale`（1）、`--reduced`、`--theme dark|light`、`--no-webgl`（每个 `getContext("webgl")` 或 `"webgl2"` 都返回 `null`，用于检查 SVG 回退）、`--preload file.js`（在页面自身脚本之前运行的脚本：计数器、桩、钩子，或 `window.__driveSpeed=0.04` 以从第一帧开始慢速运行）和 `--actions file.json`。选择器默认为第一个 `.iso-plate`，分数在元素上从 0 到 1。

`--theme`（两个脚本中都有）仅模拟 `prefers-color-scheme`。Kit 页面在构建时选择主题（`<body>` 上的 `data-theme`）并忽略该媒体查询，因此要检查浅色主题，请使用 `--light` 构建并驱动 `-light.html` 文件。

| 操作 | 作用 |
|---|---|
| `["scroll", sel]` | 将元素滚动到视口中间。 |
| `["wait", ms]` | 等待。 |
| `["move", sel, fx, fy]` | 以 10 步将鼠标滑到那里，触发 `pointermove`。 |
| `["click", sel, fx?, fy?]` | 点击，默认在中心。 |
| `["drag", sel, fx0, fy0, fx1, fy1, steps?]` | 在一点按下，以 `steps`（20）步滑到另一点，每步相隔一帧，然后释放。这些是真实输入事件（指针 id 1），因此调用 `setPointerCapture` 的处理程序可以工作。用它测试拖动旋转：从 `eval` 派发的 `PointerEvent` 没有活动指针，`setPointerCapture` 会抛出 `NotFoundError` 并中止处理程序。 |
| `["key", key, sel?, code?]` | 聚焦 `sel`（默认 `[tabindex="0"]`）并按键：`"i"`、`"Escape"`、`"ArrowRight"`、`" "`、`"7"`。数字发送 `DigitN`，字母发送 `KeyX`，标点发送其正确的键码（`Space`、`Minus`、`Comma`……）。对于共享名称的键，传入 `code`：`["key", "Shift", null, "ShiftRight"]`、`["key", "Enter", null, "NumpadEnter"]`。 |
| `["slow", factor]` | 以 `factor` 倍实时运行页面的 `requestAnimationFrame` 时间戳和 `performance.now`：`["slow", 0.05]` 是 20 倍慢动作，因此接触表可以在许多帧中捕捉 100 毫秒的事件。`["slow", 1]` 恢复实时。 |
| `["until", js, timeoutMs?]` | 每 25 毫秒轮询一个 JS 表达式，直到它为真（默认 10 秒），如果超时则打印一行。 |
| `["eval", js]` | 在页面中运行 JS 并打印结果。Promise 会被等待。 |
| `["shot", out, sel?, scale?, [x,y,w,h]?]` | 截图一个元素，或其在 CSS 像素中的区域。 |
| `["sheet", out, every, count, sel?, scale?, [x,y,w,h]?, columns?]` | 每隔 `every` 毫秒截取 `count` 帧，并将它们平铺成一张带编号的 PNG。 |
| `["turn", value, group?]` | 3D 图形：设置 `window.__isoTurn`（数字设置第一个旋转组，`group` 命名另一个，对象设置多个）并等待两帧。 |
| `["orbit", out, from, to, step, sel?, scale?, [x,y,w,h]?, columns?]` | 3D 图形：第一个旋转组各角度的接触表，每帧都标注。`["orbit", "swap.png", 129.7, 130.2, 0.1, null, 6, [x, y, w, h]]` 是一次顺序变化的 6 倍特写。 |
| `["lines", out?, sel?, { tolerance }?]` | 下面的线端检查，在之前操作留下的任何姿态下进行（分离、笔画中）。`out` 写出卡片，每个浮动端都被圈出。失败会使运行以 1 退出。 |

陷阱：
- **`scroll` 后截图是正确的**，因此你可以滚动到页面下方的图形。视口足够高以容纳整张卡片（`--height 1300`）仍然是最简单的：图形在滚动 0 处可见，其 `IntersectionObserver` 会启动它。
- **接触表中的帧间距是 `every` 加上截图时间**（60–200 毫秒，在重度 WebGL 页面上更多，此时截图也会阻塞动画帧）。对于比这更快的过渡，在接触表之前放 `["slow", 0.05]`，之后放 `["slow", 1]`。`wait`、`every` 和页面的 `setTimeout` 计时器保持实时，因此在 0.05 时每 100 毫秒的接触表相隔 5 毫秒的页面时间，而空闲巡览的恢复计时器仍按墙钟触发：按键只能将其推迟约 3.6 秒实时时间，因此在每个放慢的接触表之前再次按一个无害的键。在加载时开始的事件需要页面从第一帧就慢速：`--preload` 一个包含 `window.__driveSpeed=0.04` 的文件。
- **慢速模式不会放慢计时器。** `setTimeout` 按墙钟运行，因此在长时间放慢的截图期间，空闲巡览的恢复计时器（Raptor 的 3.6 秒 `quietTour`）可能会重启巡览并在接触表中间关闭引擎。在每个放慢的接触表之前停止巡览并等待静止：为该计时器预加载一个桩（`const st = window.setTimeout; window.setTimeout = (f, d, ...a) => (d === 3600 ? 0 : st(f, d, ...a));`），按 `Escape`，`["until", "document.querySelector('[data-readout]').textContent.includes('cold')"]`，然后才按启动过渡的键。一个标为 "opening" 但从已经打开的巡览开始的接触表会拍摄到关闭过程。
- **等待阶段，不要猜测。** `["until", "document.querySelector('[data-readout]').textContent.includes('main stage')", 8000]` 在读数到达该阶段时立即返回。
- **`null` 选择器是默认值。** 在每个操作中，缺失或 `null` 的选择器意味着第一个 `.iso-plate`（对于 `key`，是可聚焦的 `[tabindex="0"]`），因此 `["sheet", "s.png", 200, 16, null, 0.5]` 拍摄卡片。
- **长 eval 字符串会破坏内联 JSON 引号。** 将任何带有 `eval` 或 `until` 的内容放入 `--actions steps.json`。
- **页面将其实时代码内联为 `<script type="module">`，** 因此顶层变量对 `eval` 不可见。改为通过 DOM 读取状态：读数文本、`aria-valuetext`、`data-*` 属性。要计数帧或每帧读取一个值，请用 `--preload` 安装一个钩子（它在页面之前运行，因此可以包装 `requestAnimationFrame` 或 `HTMLCanvasElement.prototype.getContext`）。

## 线端
审计看到的是记录的形状，从不是线条标记，因此它无法看到在面中间停止的接缝、标线、肋条或刻度。`scripts/lines.mjs` 在无头 Chrome 中检查绘图本身：

```sh
node scripts/lines.mjs page.html --out shots/lines.png
node scripts/lines.mjs page-light.html --out shots/lines-light.png --reduced
node scripts/drive.mjs page.html '[["key"," "],["wait",2000],["lines","shots/lines-apart.png"]]'
```
```
line ends: 3 of 1148 ends on 295 lines float
  iso-line faint in it k -0.080  end  7.25 px from anything   at card 507.4,407.2   length 75.7   d M392.1 381.7L507.4 407.2…
```
卡片中每个 `.iso-line` 的每个开放子路径（来自 `fadedSvg` 的接缝、凹槽、肋条，标线、刻度、指针、导线）必须在另一个笔画、一个点或一个轮廓的 1 个 viewBox 像素内结束，或在其后绘制的填充之下。一个短的直线标记（最多 7 像素：一个刻度）需要一端在 2 像素内扎根；两端都浮动则失败。Kit 自身的轮廓、折痕、倒角和管边缘不检查，`fadeLineSvg` 渐变淡出的端也不检查。`lineSvg(d, { free: true })`（或 `"start"`、`"end"`）标记一个有意停止的端：代表印刷文本的标线、针尾、指针尖端。不要用于其他任何东西。它以卡片像素打印每个浮动端，可直接用于 `capture.mjs --zoom`，并带有其 `d` 的开头，以便你在构建中找到它；`--out` 在卡片的 2 倍截图上圈出它们。在两种主题和图形静止的每个姿态下运行它，并修复每一行直到它打印 0：

| 你看到 | 修复 |
|---|---|
| 盒子上从角落内缩的接缝或焊接线 | 使用 `k.sideSeam(plan, z, P, steps)` 作为板的 `inner`：它环绕角落并在轮廓上结束（`craft.md`，第 3 节）。 |
| 一个面上两端都内缩的加强筋或面板线 | 让它边到边运行：从面的底部到顶部，或从端到端。 |
| 肋条未到达其接缝 | `ribsOf(…, { seams: true })`，或带上的 `s0`、`s1`。 |
| 刻度浮动在裸露的板上或悬挂在边缘下方 | 将它们放在尺条上，扎根在其边缘（`topTicks` 的 `edge`，`sideTicks` 的 `top` 在面的顶边上）。 |
| 通风口或格栅线 | 闭合槽：一个细的圆角轮廓，不是线。 |
| 一条线在零件下运行并在刚过它处停止 | 在零件之前绘制它，全长，让零件隐藏它。 |
| 标签板上的标线 | `free: true`。它们代表文本。 |

## 审计
截图捕捉你想到要看的东西。带有管道、电缆和圆形零件的图形还需要检查几何形状，因为穿过管道的电缆或绘制在其前方物体之上的零件在 1 倍下不可见，在 4 倍下却很明显。将 `kit/audit.mjs` 接入构建（`kit.md`，第 12 节）：辅助函数放入一个 `A.recorder(P, { order, spring })`，构建调用 `A.settle(R, P)` 并以 `A.auditOrExit(R, P)` 结束，然后运行：

```sh
node examples/raptor-engine/build.mjs --audit
```
```
clearance: 0 problems
clearance apart: 0 problems
terminals: 0 ends not on a mount
loose ends: 0
screen folds: 0
support: 0 unsupported solids · 0 hanging tubes · 0 sunk pairs
depth order: 0 problems
through in mid-flight: 0 pairs
self-overlap: 0 chunks whose outline crosses itself on screen
wobble: 0 smoothed profiles whose slope turns between its ends
saddles off their host apart: 0
stand-in shapes: 0 items whose recorded shape covers too little of what it draws
crowding: 0 pairs where a bend or end touches another line on screen
free ends apart (marked free, not failing): 35
audit · 80 tubes · 267 solids · 478 items · 1119 depth states · 1707702 overlaps · 4292 order rules · 151.9 s
audit passed
```
它打印 `audit passed` 或 `audit failed: n problems`，并在任何失败时以非零退出。每一行都计数，包括飞行中发生的情况和 `items with no shape`，除了两个供阅读的列表：`in passing` 和 `free ends apart`。深度顺序和穿透（管对所有东西，以及实体对实体）都在相同的重放帧上检查：弹簧打开和关闭直到静止的每一帧，每个交错边界两侧 8 毫秒的反转的每一帧（`reversalsOf`：Raptor 的 35 毫秒交错上为 0.027、0.043、0.062、0.078 … 0.393 秒），以及前缀状态（`prefixStates`）。深度使用这些帧的每四分之一。如何阅读和修复每一行：

| 行 | 含义 | 修复 |
|---|---|---|
| `through A × B  clearance -2.4` | 管 A 在该弧长跨度上与 B 相交。 | 将 A 绕 B 改道，留出约 1.5 个半径的余量，或移动 B。只有 A 的端缘所在的形状被豁免，且仅在该端约一个半径内。 |
| `close A × B  clearance 0.2 < 0.6` | A 经过 B 时余量小于其半径的一半。 | 微调路线；加宽线束通道间距。 |
| `clearance apart: through share 1 …` | 在分离状态下，一根管穿过某物。 | 更改该装配体的爆炸向量。 |
| `terminals: A end 1.40 from B (rim -1.1…1.3)` | A 的端，或其端法兰的边缘，未坐在其安装座上：它浮动、倾斜，或法兰比其下方的凸台宽。 | 让路线正好在表面上结束，沿表面法线；使凸台至少与法兰一样宽；在曲面上将法兰坐在鞍垫上。电缆从不在夹具上结束。 |