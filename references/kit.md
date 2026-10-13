# Kit API

所有内容都在 `kit/iso-kit.ts` 中，`kit/iso-kit.mjs` 中提供相同的 API。世界坐标为 `[x, y, z]`，其中 x 和 y 在地面上，z 向上，单位约为 1 px。屏幕坐标为 SVG viewBox 中的 `[sx, sy]`。路径以 SVG `d` 字符串形式返回，四舍五入到 0.1。

## 目录
1. 类型
2. 相机
3. 实体
4. 面上的线条
5. 零件库
6. 移动物体
7. SVG 字符串（无框架依赖）
8. 卡片和页面
9. React 组件
10. 任意轴上的旋转零件（`kit/lathe.mjs`）
11. 管道、管子和线缆（`kit/tube.mjs`）
12. 审计（`kit/audit.mjs`）
13. WebGL 层（`kit/gl.mjs`）
14. 工具
15. 旋转组（`kit/turn.mjs`、`turn-build.mjs`、`turn-audit.mjs`）

## 1. 类型
```ts
type Flat = [number, number];            // 屏幕点
type Point3 = [number, number, number];  // 世界点
type Plane = [number, number];           // 地面点 (x, y)
interface Plan { x: number; y: number; w: number; d: number; r: number }   // 地面上的圆角矩形
interface Projection { origin: Flat; scale: number; azimuth?: number; elevation?: number }
interface SolidPaths { fill; outline; crease; top; shades: string[4]; bevel?; seamless? }
type Tone = "hi" | "mid" | "lo" | "faint";
type LineTone = Tone | "lit" | "red" | "green" | "blue";
```

## 2. 相机
| 函数 | 作用 |
|---|---|
| `fitProjection(points: Point3[], width, height, { pad = 24, headroom = 0, azimuth = 45, elevation = 30 })` | 一个投影，将这些点居中并适配到 viewBox 中。传入每个极值点，例如 `boxCorners(plan, z0, z1)` 和最高的尖端。 |
| `iso(point, P)` | 世界坐标转屏幕坐标。 |
| `depthOf(point, P)` | 值越大表示越靠近观察者。按升序排序以从后向前绘制。 |
| `towardViewer(P)` | 指向观察者的地面方向，`[cos a, sin a]`。法线指向此方向的面是可见的。 |
| `screenRise(h, P)` | 高度 `h` 覆盖多少屏幕像素。 |
| `cameraOf(P)` | 原始的正弦和余弦值。 |

## 3. 实体
所有实体都是棱柱：一个地面轮廓垂直向上拉伸。

| 函数 | 作用 |
|---|---|
| `slabOf(plan, z, h, P, steps = 8, bevel = 0)` | 一个圆角盒子。`steps` 是每个角的点数（小零件用 2–4，大圆角零件用 8–10）。`bevel`（0.3–1.6）沿可见边缘在顶部添加一条内缩的高光线，延伸至轮廓角。 |
| `cylinder(cx, cy, r, z, h, P, steps = 32, bevel = 0)` | 一个圆柱柱、圆盘、脚、旋钮或垫。小的用 16–20 步，大的圆盘用 56–72 步。 |
| `extrude(ring: Plane[], z, h, P, { convex = true, bevel })` | 任意地面轮廓。对于凹轮廓传入 `convex: false`（填充变为面而不是凸包）。 |
| `ringOf(plan, steps)` | 圆角矩形轮廓的点。 |
| `roundedPlan(plan, steps)` | 同上。 |
| `circleRing(cx, cy, r, steps)` | 圆的轮廓点。 |
| `insetPlan(plan, by)` | 缩小一个 plan（使用负的 `by` 来放大）。保持半径合理。 |
| `corners(plan, inset)` | 从角内缩的四个地面点，用于脚和螺丝。 |
| `boxCorners(plan, z0, z1)` | 八个角，用于 `fitProjection`。 |
| `haloOf(plan, z, h, P)` | 柔和阴影的填充路径：`<path class="iso-halo" d=…>`。 |

`SolidPaths` 包含的内容：
- `fill`：纸色，遮挡后面的内容。
- `shades[0..3]`：可见的侧面，按面向光源的方式分桶。
- `top`：顶面，更亮。
- `crease`：顶面与可见侧面相交处。
- `bevel`：可选。
- `outline`：轮廓。

`solidSvg` 和 `<Solid>` 按此顺序绘制它们。

## 4. 面上的线条
| 函数 | 绘制内容 |
|---|---|
| `onTop(points: Plane[], z, P, closed?)` | 水平面上的折线。 |
| `lineOnTop(a, b, z, P)` | 水平面上的一段。 |
| `planOutline(plan, z, P, steps)` | 面上的圆角矩形轮廓，用于凹槽和镶嵌（`insetPlan`）。 |
| `ring(cx, cy, r, z, P, steps)` | 面上的圆，用于螺丝环、边框和座。 |
| `segment(a, b, P)` | 任意 3D 线段。 |
| `rise(at, z0, z1, P)` | 垂直线段。 |
| `pointsAt(points, P)` | 投影一组点，用于螺丝和铆钉点。 |
| `dotGrid(x, y, cols, rows, gap, z, P)` | 投影的穿孔。 |
| `topMatrix(at, z, P, along = "x")` | 一个变换，将文本平放在顶面上，用于 `faceTextSvg`/`FaceText`。 |
| `sideMatrix(at, z, P, "left" \| "right")` | 同上，但在垂直面上。 |

## 5. 零件库
| 函数 | 返回 |
|---|---|
| `knurl(cx, cy, r, z0, z1, count, P, inset = 0.4)` | 圆柱可见半部分的垂直线。 |
| `sideArc(cx, cy, r, z, P, steps)` | 圆柱侧面上水平圆的可见前半部分：接缝和条带。 |
| `sideSeam(plan, z, P, steps = 8)` | 盒子同上：高度 `z` 处圆角 plan 的可见近半部分，环绕其角并精确终止于板坯的垂直轮廓边缘。传入板坯自己的 `steps`，并将其绘制为板坯的 `inner`。这是在盒子上绘制焊缝、条带、盖缝或装饰线的方法：绝不要使用从角内缩的 `segment`。 |
| `ringSeam(ring, z, P)` | 任意地面轮廓的 `sideSeam`（你传给 `extrude` 的环）。 |
| `sideRing(center: Point3, r, "x" \| "y", P)` | 垂直面上的圆：孔、杆端、透镜。 |
| `coil(from, to, [cy, cz], r, turns, "x" \| "y", P)` | `{ back, front }` 弹簧两半，以便你可以按后 → 杆 → 前的顺序绘制。 |
| `topTicks(from, to, step, every, edge, z, [minor, major], P, axis)` | 顶面上的 `{ minor, major }` 标尺刻度，从 `edge` 线沿 +y（或 +x）方向生长。 |
| `sideTicks(from, to, step, every, face, top, [minor, major], P, axis)` | 沿垂直面下垂的刻度。 |
| `radialTicks(cx, cy, r, count, every, z, [minor, major], P, from = 0, span = 2π)` | 表盘和转盘刻度。角度 0 指向 −y，从上方看顺时针增加。 |

## 6. 移动物体
| 函数 | 作用 |
|---|---|
| `translateAlong(P, [dx, dy, dz])` | 一个 `"translate(x y)"` 字符串，按世界偏移移动预构建的组。用于滑架、抽屉、柱塞和升降机。 |
| `holdStill(element, [dx, dy])` | 给 `element` 内的每个 `[data-hold]` 元素相反的 translate，这样当它所在的组按 `[dx, dy]` 屏幕单位移动时，夹子保持在原位。每当你移动一个包含 `G.rodOut` 夹子的项目时，用相同的偏移调用它。 |
| `axisVector(P, [1, 0, 0])` | 一个世界单位的屏幕向量。用它来将指针移动转换为世界移动。 |

绕 z 轴的旋转不能用屏幕变换完成：为每一帧或每一步重建旋转路径。对于针、指针或小零件来说这很廉价。

## 7. SVG 字符串（无框架依赖）
| 函数 | 说明 |
|---|---|
| `solidSvg(paths, { tone = "hi", crease = "faint", halo, lit, flat, className, inner })` | 当设置了 `paths.seamless`（平滑车削着色、`sphereOf`）时，组会获得 `data-seamless`，`ISO_CSS` 用各自的颜色为每个色调区域描边，宽度 0.4，这样许多相邻的色调片不会显示细线接缝。`inner` 是放在轮廓（`iso-edge`）之前的额外线条标记，用于必须位于轮廓描边下方的接缝、焊缝和条带线；React 的 `Solid` 将其作为 prop 接收。 |
| `lineSvg(d, { tone = "lo", dotted, dashed, flow, className, free })` | `flow` 动画化虚线；用于沿路径移动的东西。`free: true`（或 `"start"`、`"end"`）写入 `data-free`，告诉 `scripts/lines.mjs` 某一端意在停在裸面上：代表印刷文本的规则线、针的尾部、指针的尖端。其他每个线端都必须落在某物上。 |
| `dotsSvg(points, { size = 0.5, tone = "mid", pulse })` | 一个点可以携带第三个值，即其不透明度（`[x, y, 0.4]`），这就是 `dotsOf(..., { fade })` 让螺栓圆渐隐而不是停止的方式。 |
| `fadedSvg(list, style)` | 绘制一组各自携带不透明度的线，`[{ d, alpha }]` 或 `[d, alpha]`，如 `ribsOf(..., { fade })` 返回的那样。线条按不透明度以 0.05 为步长分组，每组在一个 `<g opacity>` 中，低于 0.02 的线条被丢弃。用于任何成员逐个向肢体渐隐的纹理：肋、鳍、滚花、规则线。 |
| `fadeLineSvg(points, { fade = [start, end], tone = "lo", className, stops = 6 })` | 一条线（屏幕点），在其前 `start` 和后 `end` 像素上渐隐而不是停止（每个渐变在其渐隐端携带 `data-free`，因此线条检查会通过它）。每个渐变沿其自身弦用 `userSpaceOnUse` 线性渐变描边（在 `stops` 步中的 smoothstep，颜色取自 `var(--anatomy-<tone>)`，因此两种主题都适用），中间是普通的 `lineSvg`，因此没有重叠的端帽、珠子或接缝。用于在面中间结束的纹理线：一条延伸出去的凹槽、一条在角前停止的焊缝、弯曲板上的规则线。保持渐变短于线条的四分之一圈，因为线性渐变沿弦。渐变 id 是路径的哈希，因此同一条线绘制两次共享一个。 |
| `wireSvg(from, to, { via = [], tone = "lo", dotted, dashed, flow, ends: "both" \| "from" \| "to" \| "none", size = 1.2 })` | 带端子点的连接器。 |
| `faceTextSvg(transform, text, { size = 3, tone = "mid", anchor })` | |
| `groupSvg(children, { class, transform, "data-part": … })` | |
| `figureSvg({ width, height, label, body, standalone: false \| "dark" \| "light", className })` | `<svg>`。`label` 是可访问描述：说明对象是什么以及它展示了什么。`className` 添加到其 class，这在你将其堆叠在画布下时很有帮助（`rt-back`、`rp-back`）。 |

## 8. 卡片和页面
| 函数 | 说明 |
|---|---|
| `plateHtml({ fig, title, hint, readout, keys: [{ mark, label }], caption, body })` | 卡片。`mark` 是 `raised`、`flat`、`lit`、`edge`、`dim`、`dotted`、`dashed`、`red`、`green`、`blue` 之一。读数元素携带 `data-readout`，因此实时代码写入它。 |
| `pageHtml({ title, theme = "dark", body, script, width = 650 })` | 一个独立文档，内联了 `ISO_CSS`。 |
| `kitScript()` | 来自 `scripts/inline-kit.mjs`。将其前置到 `script`，以便页面在动画时可以调用 kit 函数，如 `lineOnTop` 和 `translateAlong`。 |
| `glScript()` | 来自 `scripts/inline-kit.mjs`。`kit/gl.mjs` 的源代码（`GLSL_ISO`、`glCamera`、`glLayer`、`smooth`、`burst`、`settle`），在带有着色器的页面上内联在 `kitScript()` 之后：`script: kitScript() + glScript() + live`。它不依赖 kit，因此实时代码不调用任何 kit 函数的页面可以单独内联它（Raptor 使用 `glScript() + LIVE`）。所有内容都是一个模块脚本，因此实时代码不得重新声明 `smooth`、`burst` 或 `settle`：第二个 `const smooth` 是 SyntaxError，会杀死页面。 |
| `turnScript()` | 来自 `scripts/inline-kit.mjs`。3D 运行时（`TURN`、`TURN_CSS`），去掉了 `export`，用于带实时零件的页面：`script: turnScript() + glScript() + live`。其内部以 `t3`/`T3_` 为前缀，因此它可以与 kit 和 WebGL 层一起内联。 |
| `ISO_CSS` | 所有样式。`kit/iso.css` 是相同的文本。 |

## 9. React 组件
来自 `kit/react/draw.tsx`，它是服务器安全的：

- `<Solid paths tone crease halo lit flat className inner />`（`inner` 是放在轮廓下方的线条标记，如 `solidSvg`）
- `<Line d tone dotted dashed flow free />`
- `<Wire from to via tone dotted dashed flow ends size />`
- `<Dots points size tone pulse />`（一个点可以携带不透明度作为 `[x, y, alpha]`，如 `dotsSvg`）
- `<FaceText transform size tone anchor>text</FaceText>`
- `<IsoFigure width height label>` …svg children… `</IsoFigure>`
- `<Plate fig title hint readout keys caption>` …figure… `</Plate>`

来自 `kit/react/live.tsx`，它是 `"use client"`：

- `useReadout()` 返回 `{ anchor, say }`。将 `anchor` 放在你的根元素上；`say(text)` 写入卡片的读数。
- `useLoop(tick: (dt, calm) => keepGoing, running)` 在 `running` 时且 `tick` 返回 true 时运行帧循环。它返回 `kick()` 来重启它。
- `follow(current, target, dt, seconds, calm)` 是指数缓动。
- `spring(state, target, dt, stiffness, damping, calm)` 是子步进弹簧。它在稳定时返回 true。
- `useEased(target, seconds)` 返回一个缓动数字作为 React 状态。适用于小图形。
- `useInView(ref)`、`useStillness()`、`asksForStillness()`。
- `Stage` 是 `IsoFigure` 使用的包装器。它为 CSS 动画添加 `data-inview`。

## 10. 任意轴上的旋转零件（`kit/lathe.mjs`）
`kit/iso-kit.mjs` 构建立在地面上的棱柱。`kit/lathe.mjs` 围绕**任意**轴旋转一个轮廓，用于钟形、腔室、法兰、阀体、泵蜗壳、管件、倾斜放置的执行器，以及垂直面上的连接器。它返回与 `slabOf` 相同的 `SolidPaths`，因此 `solidSvg` 绘制它们。将其导入为 `import * as G from "../../kit/lathe.mjs"`。

**框架。** 框架是 `{ o, a, u, v }`：一个原点、单位轴和两个单位径向方向。零件上的一个点是 `(s, r, angle)`：`s` 沿轴从 `o` 起，`r` 从轴起，`angle` 从 `u` 朝向 `v`。

| 函数 | 作用 |
|---|---|
| `frameAlong(o, a)` | 通过 `o` 在轴 `a`