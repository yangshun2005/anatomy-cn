# 运动与交互

当一个图形的机构像真实物体那样运动时，它就活了起来：滑架滑行并停稳，弹簧过冲一次，指针摆向新的读数。运动应当足够平缓以便跟随，并且忠实于模型。

## 原则
- **基于时间。** 从 `requestAnimationFrame` 时间戳中获取 `dt`，并将其限制在 1/30 秒以内，这样后台标签页就不会跳变。绝不要每帧步进固定量。
- **向目标缓动。** 指针和键盘设置一个*目标*，部件通过指数缓动（`x += (target − x)·(1 − e^(−dt/τ))`，τ = 0.1–0.3 秒）或弹簧跟随。部件绝不直接吸附到光标。
- **在主体具有弹性时使用真实弹簧。** 使用真实模型中的刚度和阻尼，每秒约 240 次子步进以保证稳定性。当误差和速度都低于 5e-4 时吸附到目标。
- **每个图形一个循环。** 单个 `tick(dt)` 更新模型，然后通过 refs 写入 DOM：`setAttribute("transform" | "d")`、`textContent`、`dataset`。不要每帧重新渲染 React。
- **在停稳或离屏时停止。** 使用 `IntersectionObserver`，rootMargin 为 `120px 0px`，并读取最新的条目（`entries[entries.length - 1]`）：当舞台离开又返回发生在两次回调之间时，一次回调会按从旧到新的顺序传递两个条目，读取第一个会让图形冻结在屏幕上。一旦所有内容都到达目标，就停止循环。
- **减少运动。** 当设置了 `prefers-reduced-motion: reduce` 时，直接跳到目标，没有巡览、流动或脉冲。图形仍必须响应输入。
- **空闲巡览。** 当无人交互时，约 1.4 秒后播放一段脚本化循环，展示图形的要点。在悬停、聚焦或按下时暂停，并在最后一次交互后约 3.6 秒恢复。巡览驱动与用户相同的目标。
- **读数实时更新**，来自同一模型，使用真实单位且简短：`block 2 · reads 14.0 px`、`120.0 × 32.0 · lean 10.6 px`。从部件所在的位置读取，而不是从目标读取：当任何弹簧正在运动或待定时，Raptor 显示 `taking apart` 或 `closing up`，只有在那之后才显示 `apart · 11 assemblies` 或 `1,630 kg · cold`。阶段词和它旁边的数字必须来自同一来源的同一瞬间：一个盲测台打印了 `end of stroke · relief 85 bar`（词来自模型，数字来自滞后 0.25 秒的压力表指针），尽管其泄压阀在 92 时开启，而在回缩后立即打印 `idle · 44 bar`。

## 指针
- 使用轴的屏幕向量将指针映射到世界空间。对于沿 x 方向的运动，使用 `axisVector(P, [1, 0, 0])`，或将图形的宽度比例映射到部件的行程范围。当行程较短时，范围映射更简单且手感更好。
- 悬停预览，按下提交。在触摸设备上没有悬停，所以让轻触执行操作，并保持 `touch-action: pan-y`，这样页面仍可滚动。
- 拖动时使用指针捕获，并在 `pointerup` 或 `pointercancel` 时释放。

## 键盘与无障碍
- 图形的根元素可聚焦（`tabIndex=0`），带有 `role="slider"`（或当存在离散选择时使用 `"group"` 加按钮）、解释控件的 `aria-label`，以及用文字表示的 `aria-valuenow`/`aria-valuetext`，例如 "Pressed: 129.6 by 34.6 pixels, leaning 10.6 px"。当读者的选择改变时（部件被选中、拆开或组装、点火或冷却）写入值文本，而不是从循环中写入：屏幕阅读器会播报聚焦滑块值文本的每次变化，因此每帧跳动的读数（"Pc 290 bar · 222 tf"）会变成一串数字流。保持视觉读数与无障碍值分离（Raptor 的 `say()` 只写入读数，`describe()` 写入值文本）；实时区域承载阶段。
- 按键：方向键按合理单位步进（Shift 用于更大步进），Home 和 End 到达极值，Space 或 Enter 按下或切换，Escape 释放。
- 显示焦点环：`outline: 1px solid var(--anatomy-muted); outline-offset: 6px`。
- `<svg>` 带有 `role="img"` 和描述对象及其展示内容的 `aria-label`。

## 原生循环（无框架）
```js
let frame = 0, last = 0, visible = false, touring = !still.matches, clock = 0;
function tick(now) {
  const dt = last ? Math.min((now - last) / 1000, 1 / 30) : 1 / 60;
  last = now;
  const calm = still.matches;
  if (touring) { clock += dt; target = tourAt(clock); }
  x = calm ? target : x + (target - x) * (1 - Math.exp(-dt / 0.28));
  springTo(tip, heightUnder(x), dt, 420, 26, calm);
  draw();                                   // setAttribute on refs, readout.textContent = …
  const settled = !touring && x === target && tip.rate === 0;
  frame = visible && !settled ? requestAnimationFrame(tick) : 0;
}
const run = () => { if (!frame && visible) { last = 0; frame = requestAnimationFrame(tick); } };
new IntersectionObserver((entries) => { visible = entries[entries.length - 1].isIntersecting; if (visible) run(); }, { rootMargin: "120px 0px" }).observe(stage);
```
`examples/dial-indicator/build.mjs` 包含一个完整版本，带有巡览、指针映射和键盘停靠点。

## React 循环
将静态图形保留在服务器上构建的 props 中，只移动 `<g>` 包装器和改变形状的路径：

```tsx
"use client";
export function RigLive(props: { base: React.ReactNode; carriage: React.ReactNode; dial: React.ReactNode }) {
  const { anchor, say } = useReadout<HTMLDivElement>();
  const car = React.useRef<SVGGElement>(null);
  const needle = React.useRef<SVGPathElement>(null);
  const state = React.useRef({ x: REST, target: REST, tip: { at: 0, rate: 0 } });
  const visible = useInView(anchor);
  const kick = useLoop((dt, calm) => {
    const s = state.current;
    s.x = follow(s.x, s.target, dt, 0.28, calm);
    const settled = spring(s.tip, heightUnder(s.x), dt, 420, 26, calm);
    car.current?.setAttribute("transform", translateAlong(P, [s.x, 0, 0]));
    needle.current?.setAttribute("d", needleAt(s.tip.at));
    say(readoutOf(s));
    return !(settled && s.x === s.target);
  }, visible);
  return (
    <div ref={anchor} tabIndex={0} role="slider" onPointerMove={(e) => { state.current.target = toWorld(e); kick(); }}>
      <IsoFigure width={W} height={H} label={LABEL}>
        {props.base}
        <g ref={car}>{props.carriage}</g>
        {props.dial}
        <path ref={needle} className="iso-line" data-tone="lit" d={NEEDLE_REST} />
      </IsoFigure>
    </div>
  );
}
```
`examples/test-rig/live.tsx` 是完整生产版本，带有弹簧、巡览、按压和拉动、键盘处理和 aria。

## 改变形状
当部件改变的是形状而非位置时（膨胀的胶囊、压缩的弹簧、转动的指针），每帧使用工具包仅重建其路径，使用 `slabOf`、`coil` 或 `lineOnTop`，并将它们写入其路径：`fill`、`shades[0..3]`、`top`、`crease`、`bevel`、`outline`。一个实体成本远低于 0.1 毫秒。测量它：一帧应保持在约 1 毫秒的脚本时间以内。
- **预计算帧步进。** 一个从 41 个预构建帧在 ±15° 范围内绘制的杠杆，其旋钮每帧移动约 0.8 px，在慢动作中可见。每帧重建旋转部件，或预计算足够的帧，使一步移动不超过 0.25 px。
- **从密封套中滑出的杆**不改变形状：它是一根全长杆在移动，由保持不动的裁剪在密封套面处裁剪（`G.rodOut`、`holdStill`、`craft.md`，第 3 节）。绝不要将其绘制为多个重叠副本。

## 一次性事件的包络
闪光、咔嗒、弹出、抖动或冲击是一个带有年龄的事件，而不是你设置的值。用具有攻击和衰减的包络驱动它。`kit/gl.mjs` 导出 `smooth(a, b, x)`（smoothstep）和 `burst(age, rise, fall)`，后者是 `smooth(0, rise, age)` 乘以峰值后的 `exp(−(age − rise)/fall)`，事件前为 0：
```js
import { smooth, burst, settle } from "../../kit/gl.mjs";
const flash = calm ? 0 : burst(clock - mainAt, 0.08, 0.13);
```
在独立页面上，`glScript()` 已经内联了这三者，所以直接使用它们，不要声明自己的：同一模块脚本中的第二个 `const smooth` 是 SyntaxError。
- 攻击为 40–120 毫秒，smoothstep 使其在起始处斜率为零。在一帧内将值设为 1 是弹出。
- 存储事件的开始时间（`mainAt = clock`），每帧根据其年龄计算每个包络。重叠的两个事件只需相加。
- 给每个事件自己的标志（`popped`），这样快速重启不会吞掉下一个，也不会切断仍在衰减的那个。
- 在减少运动下，每个事件包络为 0。

## 临界阻尼停稳
对于应当无过冲到达的值（色相、亮度、灯、伸展、频率），使用工具包的 `settle(state, target, dt, seconds, calm)`：在 `state = { x, v }` 上的临界阻尼弹簧，时间常数为 `seconds`，`v += (w²·(target − x) − 2w·v)·dt`，其中 `w = 1/seconds`，分四个子步。它比指数跟随更平滑，因为速度也是连续的。当误差和速度都低于 1e-4 时吸附到目标，在 `calm` 下直接跳到那里，并在仍在运动时返回 true，因此它可以保持循环存活：
```js
const moving = settle(freq, state.tf, dt, 0.625, calm);
```
波纹水槽通过 `settle` 运行其频率（0.625 秒）和浸渍器间距（0.4 秒），因此按键或指针移动绝不会跳变波长。
- **分离阶段所改变的内容。** 当一个物理量（比如腔室压力）驱动颜色、形状和亮度时，该量上的快速弹簧会让三者同时跳变。给颜色自己的较慢 `settle`（0.3–1 秒），并计算亮度使其在变化过程中单向移动。
- **淡出只会变暗。** 当停止开始时，将亮度冻结在当前水平，并仅按下降的量缩放它。
- **改变一个弹簧会改变其邻居。** 临界阻尼一个弹簧会减慢所有依赖其旧过冲的阶段，因此重新检查所有阶段。

## 爆炸视图
悬停时拆解机器是展示其部件的强有力方式（`examples/raptor-engine/`）：
- **在组装位置构建每个部件**，标记装配名称，并给每个装配一个世界移动向量（`PIECES.nozzle.move = [0, 0, -120]`）。移动遵循真实装配：喷嘴下落最远，侧单元径向滑出（`out(deg, r, z)`），支架不动。
- **每个装配一个弹簧**，组装时值为 0，拆开时为 1。Raptor 使用刚度 64 和阻尼 14.2，刚好低于临界，因此部件停稳时带有一丝重量感，每帧子步 4 次。
- **错开弹簧。** 打开时，每个装配的目标在前一个之后 35 毫秒改变，顺序让任何东西都不会穿过任何东西（Raptor：`nozzle, downcomer, controller, plumbing, lox, ftp, hotgas, chamber, orbp, otp, gimbal, stand`，见下文 "Order the stagger"）。关闭使用相反顺序，因此机器从内到外折叠回去。将待定目标保留为 `{ value, at }`，并在 `clock >= at` 时应用它们。
- **在反转时，只重新调度变化的内容。** 当读者在中途反转时（快速悬停、Space、轻触），取消每个尚未离开的装配的待定目标，并仅按其在较短列表中的索引错开目标变化的那些。最后一个出去的然后立即成为第一个回来的。重新调度整个列表会让早期装配向拆开方向飞出 0.3 秒，而后期装配留在原位，爆炸之所以有效只是因为下一个装配在 35 毫秒后移开：在该窗口内的反转会将热气管路撞入尚未离开的甲烷泵。使用此规则，沿错开的份额在每一帧都保持顺序（较早的装配总是领先），这是下面前缀规则所依赖的。
- **使用变换移动。** 每个项目的屏幕偏移是其移动向量，投影并乘以弹簧值：`translate(tx ty)`。在 0 时移除属性，这样组装图形正好是静态图形。
- **保持线条落位。** 连接两个装配的部件（框架和泵凸耳之间的执行器，或盒子和插头之间的电缆）属于它们物理上保持在一起的装配，并且拆开时它们仍终止于某物上。电缆两端获得端子点。
- **选择。** 悬停、键盘方向键和部件列表都选择一个装配。选中的部件成为唯一的 `lit` 强调：`.it[data-on]` 上的 CSS 将 `hi` 移到 `lit`，将 `mid` 移到 `hi`，并将顶部和侧面切换为 lit 填充。其他每个部件通过 `[data-focus]` 上的 CSS 降低一个色调（`hi` 到 `mid`，`mid` 到 `lo`），读数说明所选部件的作用。

## 移动部件的深度重排序
向观察者移动或远离观察者的部件在移动时改变画家顺序。当它们这样做时，每帧排序：
- 给每个项目一个静态键 `k = dot3(ref, viewOf(P)) + bias`，其中 `ref` 是其中心，`bias` 解决接触部件之间的平局。同时给它一个提升：`lift = dot3(move, viewOf(P))`。
- 每帧，其键为 `k + lift × spring`。按该键对索引排序，并**仅在顺序实际改变时重新追加 DOM 节点**（一个 `DocumentFragment`）。大多数帧不改变任何内容。
- 将键和移动向量存储在元素上（`data-k`、`data-m`），这样实时脚本不需要几何信息。
- 管件按片段排序（`tubePieces` → `piece.mid`），因此一根管道可以穿过一个部件前面和另一个部件后面。
- **从运动中确定静态键，而不仅仅是从静止姿态。** 每个项目的键是 `k + lift × spring`，但在错开弹簧的情况下，每个装配在任意帧都有自己的份额，因此两个在组装和拆开时顺序正确的项目在中间仍可能被画反：在第一个 Raptor 中，一条增压管线在每次爆炸的 40% 时间里隐藏在其前方的执行器柱后面。Raptor 在 0、0.1 … 1 处测量每个重叠对的真实深度，在其自身弹簧的每第四帧处，用实时错开重放直到它们停稳，打开、关闭以及在每个错开边界两侧反转，并在前缀状态下（`A.explodeStates(STAGGER, SPRING).depth`，即 `[...A.uniformStates(10), ...A.springStates(STAGGER, { ...SPRING, stride: 4, reversals: A.reversalsOf(STAGGER, SPRING.stagger) }), ...A.prefixStates(STAGGER)]`，1,119 个状态，以及用 `{ order: STAGGER, spring: SPRING }` 创建的记录器的 `A.settle(R, P)` 默认值），`orderKeys` 一次性解决所有这些问题（每条规则使用该帧的份额）。仅打开和关闭会在快速悬停产生的帧中留下错误对：一个应位于腔室带前面的吹扫法兰在 0.20 秒的反转时消失在其后面。审计在静止时和任何重放帧中对任何错误对失败；只有无人看到的均匀状态可以持有互锁。
- **弹簧的单一来源。** 将错开顺序和弹簧常量（`SPRING = { stagger, stiffness, damping, substeps }`）保留在构建中，并将其传递到页面的数据中，这样审计重放的正是查看者所看到的。
- **用部件所安装的东西移动它（`ride`）。** 螺栓固定在腔室上的传感器随管路高亮，但随腔室的弹簧移动。每个项目在 `piece`（它随其高亮的装配）旁边记录 `ride`（其弹簧移动它的装配，从其移动向量找到），页面读取 `data-ride`，`place()` 使用 `piece[node.ride].x` 进行平移和排序。借用另一个装配的向量但保留自己弹簧的项目会沿其安装座滑动，并在