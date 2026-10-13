# 绘图中的 WebGL

如何将着色器效果（火焰、水、光、辉光）放入等距图形中，使其看起来像是同一物体的一部分，并通过同一相机观察。有两个已发布的示例做到了这一点：`examples/raptor-engine/`（火箭发动机下方的羽流、激波、飞溅和水蒸气）和 `examples/ripple-tank/`（焦散、阴影和波纹水面）。请将它们的 `live.js` 与本页对照阅读。

用户很喜欢第一个着色器图形，但注意到它的颜色在各阶段之间瞬间变化，就好像效果被切断了一样。所以标准是：**没有突变，没有切断，一切都处于绘图的透视之中。** 下面的每一条规则都来自一个打破了这一标准的 bug。

## 目录
1. 着色器何时才有存在的价值
2. 图层堆叠
3. 相机契约
4. 等距相机中的光线步进
5. 将着色器内容与 SVG 对齐
6. API 与最小页面
7. 工艺规则，每条都附有教会它的 bug
8. 浅色主题（8b. 旋转组上的着色器）
9. 验证
10. 性能

## 1. 着色器何时才有存在的价值

对于线条无法绘制的东西使用着色器：发出的光和辉光、火焰和尾气、水面、焦散、真实灯具的柔和阴影、烟雾和水蒸气。这些是连续的场，它们要么是主体本身，要么是主体的行为。

不要用于：
- **实体。** 零件、边缘、刻度线和螺丝保持使用套件中的 SVG。着色器绝不绘制 SVG 可以绘制的零件。
- **装饰。** 闪烁、背景渐变、每条边缘上的泛光，或为了让图形看起来"丰富"而添加的噪声。如果你可以删除该效果而解释仍然成立，那就删除它。
- **SVG 已经能很好展示的东西。** 发光的 LED 是一个 `lit` 圆点，而不是着色器。

SVG 必须仍然能够独立阅读。没有 WebGL 的 Raptor 仍然展示发动机，带有边缘光和 CSS 甲板辉光。波纹水箱将其波峰和节线绘制为 SVG 线条（`model.mjs` 中的 `patternOf`），裁剪到水箱在纸上的阴影的绘制轮廓内，并在 WebGL 运行后通过 `.rt-stage[data-gl] .rt-fallback { display: none }` 隐藏它们。

## 2. 图层堆叠

着色器无法与 SVG 进行深度测试，所以你要改为对图层排序。基本的堆叠是在一个 `position: relative` 舞台中的三个图层，全部具有相同的 viewBox：

```
back SVG     效果覆盖的内容        (k.figureSvg: role="img" 和标签)
canvas       效果                  (position:absolute; inset:0; pointer-events:none)
front SVG    覆盖效果的内容        (aria-hidden)
```

对于每个零件，问一个问题：**它是否会被效果覆盖，或者它是否覆盖效果？** 被覆盖的零件放在 back SVG 中，覆盖的零件放在 front SVG 中。如果一个零件两者兼具，你有三个选择：

- **拆分图形。** 将零件切成碎片，把每个碎片送到它所属的图层以及该图层中的槽位。波纹水箱的电机电缆是三组 `tubePieces`：跨过桥的部分放在 front SVG 中横杆之前，沿近侧边缘的部分放在 front SVG 中近侧端盖之后，沿左腿向下到控制箱的部分放在 middle SVG 中。
- **在着色器中建模。** 如果零件是简单形状（盒子、圆柱、圆环）并且留在 back SVG 中，将该形状作为遮挡体提供给着色器。Raptor 的立柱是不透明盒子（`columnNear`），所以每个空气积分都从 `max(tGround, tColumn)` 开始。喷淋环是 `groundHit` 内的一个圆环，所以飞溅照亮其内侧，而外侧保持黑暗。
- **拆分效果。** 当效果从 front-SVG 零件前方经过时，在 front SVG 上方添加第二个 canvas。遮挡零件的解析代理为每个采样决定它去 back canvas（权重 `1 − f`）还是 front canvas（权重 `f`）。每个采样只绘制一次，所以没有接缝。让代理宽松一些：在真实轮廓之外，两个图层看起来相同。

### 实例：Raptor
```
back SVG    甲板、支脚、火焰锥、喷淋环、立柱、梯子、电缆桥架
canvas      .rp-plume: 羽流体积、激波（后半部分）、闪光、飞溅、地面环、发动机后方的水蒸气
front SVG   每个发动机项目（按深度排序的 .it 组）、管道、线束、十字头
glow SVG    .rp-glow: 两个喷嘴唇缘，屏幕混合，所以它们只能增加光
canvas      .rp-veil: 位于发动机前方的水蒸气和激波采样
div         .rp-flash: 用于点火闪光的 CSS 径向渐变（closest-side）
```
发动机代理是 `engineR(z)`，一个关于钟形、燃烧室和泵的半径函数。`inFront(p, V)` 说明采样 `p` 是否位于发动机和观察者之间。`side(front, veil)` 将采样发送到一个或另一个 canvas。

### 实例：波纹水箱
```
back SVG    底板、支脚、标尺、标签板、带胶带的纸张、跨过底座边缘的灯电缆、SVG 回退图案
canvas      .rt-paper-gl: 纸和底座上的灯光、焦散、支脚、柱、桥、电机、控制箱和泡沫海滩的阴影、折射
middle SVG  柱、臂、灯及其电缆、控制箱、沿左腿在夹子中的电机电缆、带调平螺丝的支脚、
            远侧导轨、三个角柱、远侧玻璃、泡沫海滩、平水填充
canvas      .rt-water-gl: 窗户反射和室内光斑、壁面处的弯月面倾斜、浸渍接触环、
            近侧玻璃后面的水带、玻璃色调、灯的光束（仅深色）
front SVG   远侧端盖、跨过桥的电机电缆、远侧支柱和弹簧、横杆、浸渍器、电机、近侧弹簧、近侧导轨、
            玻璃底部、水线、近侧玻璃、近侧端盖、边缘上的电机电缆、近侧角柱、近侧支柱、横梁
```
两个 canvas 位于两个深度，因为有两个高度上的两个效果：纸上的光（支脚下方）和水面（水箱内部）。控制箱立在底座上、水箱旁边、水 canvas 下方，所以它位于 middle SVG 中；近侧角柱在它所站立的端盖之后绘制。

堆叠的 CSS（改编自 `ripple.css`）：
```css
.fx-stage { position: relative; touch-action: pan-y; }
.fx-stage > svg, .fx-stage > canvas { display: block; width: 100%; height: auto; }
.fx-stage > .fx-layer { position: absolute; inset: 0; height: 100% !important; pointer-events: none; }
.iso:not([data-theme="light"]) .fx-gl { mix-blend-mode: screen; }
```
屏幕混合只能提亮。一个还必须使其覆盖的内容变暗的 canvas（波纹水箱的水，它给透过它看到的纸着色）在深色中也保持正常预乘混合：只有 `.rt-paper-gl` 使用屏幕混合。
如果 front SVG 必须接受悬停（Raptor 的零件会），给它 `pointer-events: none`，给它的项目 `pointer-events: visiblePainted`，这样 canvas 和空白区域永远不会捕获指针。

## 3. 相机契约

`kit/gl.mjs` 将 `GLSL_ISO` 前置到你的片段着色器。它声明了使着色器看到与 `iso()` 相同相机的 uniform 和函数：

| 名称 | 它是什么 |
|---|---|
| `uCanvas` | 设备像素中的 canvas 尺寸。 |
| `uView` | viewBox 尺寸 `[W, H]`。 |
| `uOrigin`、`uK` | 投影原点和每世界单位的屏幕单位（`glCamera(P).k`，等于 `scaleOf(P)`）。 |
| `uCam` | 方位角和仰角的 `(sinA, cosA, sinE, cosE)`。 |
| `uTime` | 你传给 `draw` 的时间。 |
| `viewOf(gl_FragCoord.xy)` | 设备像素到 viewBox 点（它翻转 y）。 |
| `project(vec3 p)` | 世界到 viewBox，与套件的 `iso()` 相同。 |
| `towardViewer()` | 单位 3D 视线方向，`(cosA·cosE, sinA·cosE, sinE)`。每个 `p + t·towardViewer()` 都落在同一像素上。与套件的 JS `towardViewer(P)` 不同，后者返回 2D 地面方向。 |
| `rayOf(vb)` | 穿过原点、面向相机的垂直平面上 `vb` 下方的世界点，所以 `rayOf(vb) + t·towardViewer()` 是该像素的光线。 |
| `pixelOf()` | 屏幕上每 canvas 像素的世界单位，`(uView.x / uCanvas.x) / uK`。 |
| `lightDir()` | 套件的光，等于 `lightOf(P)`，所以着色器着色与 SVG 色调一致。 |
| `onFloor(vb, z)` | 此像素下方高度 `z` 的水平平面上的世界点。 |
| `onWallX(vb, x)`、`onWallY(vb, y)` | 此像素下方垂直平面 `x = const` 或 `y = const` 上的世界点。 |
| `hash12`、`hash13`、`noise2`、`noise3`、`fbm2`、`fbm3` | 哈希和值噪声，加上 4 倍频程 fbm。 |
| `window(x, a, b, soft)` | 在 `[a + soft, b − soft]` 内为 1，在 `[a − soft, b + soft]` 外为 0：每个边界上以 `2·soft` 宽的 smoothstep 为中心，所以在 `a` 和 `b` 处为 0.5。 |
| `tonemap(c, exposure)` | `1 − exp(−c·exposure)`。用它代替 `clamp`。 |

`onFloor` 和墙面函数与 `iso()` 精确往返：`project(onFloor(vb, z)) == vb`。这正是让着色器内容与 SVG 线条对齐的原因。

**为什么用世界空间。** 图形的透视就是投影。如果你在屏幕空间中绘制效果（凭眼睛确定大小的椭圆、以像素为单位的辉光柱），它无法与绘图一致。它会漂浮在地面上方、穿过甲板，或者在相机变化时保持相同形状。一切都从世界点和世界尺寸构建。屏幕尺寸只来自世界单位乘以 `uK`。

世界单位中的抗锯齿：`pixelOf()` 是屏幕上每设备像素的世界单位。在深度方向的地面上，将其除以 `uCam.z`（sinE），在墙面上除以 `uCam.w`（cosE）。将已对齐的边缘柔化约 `0.7 * px`，这样它们会呈现一个像素的柔和：像 SVG 一样清晰，但没有锯齿。在折射拉伸图像的地方，固定的柔化会产生阶梯；波纹水箱启用 `GL_OES_standard_derivatives`，并通过 `length(fwidth(q.xy)) / length(fwidth(rest))` 加宽其阴影半影，即偏移点的足迹与未偏移点的足迹之比，在静止时恰好为 1，并带有 `#ifdef GL_OES_standard_derivatives` 回退。

## 4. 等距相机中的光线步进

相机是正交的，具有均匀缩放：投影的两行都是正交归一的。由此得出三件事。

- **每个像素都是一条直线光线** `p(t) = p0 + t·V`，其中 `V = towardViewer()`，`p0` 是像素下方的任意世界点。取 `p0 = onFloor(vb, h)`，或套件的 `rayOf(vb)`，即穿过原点、面向相机的垂直平面上的点（Raptor 的光线状态从那里开始：`S.g0 = rayOf(vb)`）。较大的 `t` 更靠近观察者。
- **到点的距离就是屏幕距离。** 屏幕距离除以 `uK` 就是从像素光线到世界点的精确垂直距离。球形闪光、激波壳或辉光球可以从该距离计算，并且仍然是世界正确的。最近接近参数是 `tc = dot(C − p0, V)`。
- **遮挡体后面的可见比例。** 如果沿光线的第一个不透明表面在 `tLow` 处，则以 `tc` 为中心、宽度为 `sigma` 的高斯可见比例为 `1/(1 + exp(2.41·(tLow − tc)/sigma))`（Raptor 的 `beyond`）。这给出了柔和的切割，但是几何上的。

要步进一个体积：
1. **找到沿光线的第一个不透明命中**：地面，加上 SVG 绘制的任何高度场。Raptor 从火焰锥尖端上方步进 28 次到甲板，然后在由与 SVG 车削轮廓相同数字构建的 `ground(r)` 高度场上二分 7 次。只有命中前方的光线部分（`t > tLow`）是空气。
2. **用简单形状界定步进**（Raptor 使用从地面命中向上到出口平面的半径 132 的圆柱）。将每一项窗口化到该边界之前恰好为零，所以边界只是优化。
3. **用抖动采样。** 使用 `t = ta + (i + hash12(gl_FragCoord.xy))·dt`，这样 28 个采样不会产生条带。
4. **将每一项按其弦长归一化**，这样无论步数多少，中心像素都获得预期的亮度。对于垂直轴体积 `r² = u² + (t·cosE)²`，所以半径为 `σ` 的高斯沿光线积分为 `σ√π / cosE`：将每个采样加权 `dt·cosE/(σ·1.772)`。
5. **预期均匀圆柱看起来更柔和。** 它的投影轮廓是 sqrt 形状的，所以它失去了平面绘图所具有的明确边缘。在深色中，添加一个边缘壳（Raptor 在 `0.9·Re` 处的橙色剪切层）来恢复边缘。在浅色中，同一个壳会在空心火焰周围勾勒出轮廓，所以在那里你改为将主体向轴加权（第 8.4 节）。

有些项不需要步进：
- 沿光线看到的薄水平圆盘是精确的椭圆。Raptor 的马赫盘保持解析：`u² + ((s0 − sc)/tanE)² < rm²`。
- 沿光线积分的薄球壳是边缘增亮的：亮度与 `rs·w / sqrt(rs² − d² + 2·rs·w)` 成正比。将归一化轮廓平方，否则内部会读作乳白色气泡。

## 5. 将着色器内容与 SVG 对齐

任何位于平面上的东西都必须与该平面的 SVG 线条精确对齐。
- **相同的数字。** 通过 `DATA` 将 SVG 的平面图和高度传给着色器（波纹水箱传递 `inside`、`outside`、`paper`、`base`、`round`、`z` 以及所有投射阴影的物体的平面图），并将它们作为 `fixed` uniform 一次性交给 `glLayer`（其 `FIXED`）。用带符号距离 `sdRect` 对相同的圆角矩形进行测试，这样被照亮的纸角会终止于绘制的圆角半径。
- **相同的平面。** 波纹水箱使用 `onFloor(vb, uZ.x)`（纸）、`onFloor(vb, uZ.z)`（水）以及 `onWallX(vb, uNear.x)` 和 `onWallY(vb, uNear.y)`（玻璃后面水带的两面近侧玻璃壁）。绝不使用屏幕偏移。
- **透过开口看到，被掩码两次。** 较近平面中开口（观察窗、窗户、凹槽、其边框下的仪表盘面）后面的内容被其自身平面（`onWallY(vb, back)`）和开口平面中测试的开口（`onWallY(vb, face)`）掩码，两个柔和掩码相乘。只在后平面中掩码时，一个盲测观察窗将其油和玻璃溢出到主体面上方边缘约 0.4 px 处，在底部偏移在开口内显示出一条裸露主体的条带。在 front SVG 中绘制凹槽的内壁，按朝向着色（`craft.md`，第 10 节），这样两个平面之间的偏移读作壁，而不是裸露的面。
- **只有偏差移动。** 如果效果位移你所看到的内容（通过波纹的折射），只应用与静止状态不同的部分。波纹水箱计算由波纹表面和平静水面弯曲的视线，并只应用差值。在静止时，水下的纸与 SVG 纸对齐。在视线到纸的路径经过支脚几个单位以内的地方（`legClear`），偏移也淡化为零，这样折射的纸永远不会从 middle SVG 在其上绘制的支脚后面滑出。
- **高度场与车削匹配。** Raptor 的 `ground(r)` 是甲板圆盘加上火焰锥轮廓，具有与 `build.mjs` 中 `G.solidOf` 轮廓相同的半径和高度。锥体后面的流动在几何上被隐藏，硬边缘只出现在 SVG 绘制真实轮廓的地方。

在 4× 下检查对齐（第 9 节）。

## 6. API 与最小页面

```js
import { GLSL_ISO, glCamera, glLayer, smooth, burst, settle } from "../../kit/gl.mjs";
import { kitScript, glScript } from "../../scripts/inline-kit.mjs";
```

| 函数 | 它做什么 |
|---|---|
| `glCamera(P)` | 套件投影的 `{ origin, k, cam }`。在 Node 和浏览器中都能工作。放入你的页面 `DATA`。 |
| `glLayer({ canvas, fragment, view, camera, uniforms = [], fixed = {}, extensions = [], maxRatio = 1.5, onError })` | 在 `canvas` 上用预乘