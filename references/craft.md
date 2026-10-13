# 工艺

一张图如何读起来像一件精心制作的物件，而不是一张示意图。这里的每一条规则都来自一张被严苛审阅者拒绝或批准的图。

## 目录
1. “精心制作”意味着什么
2. 反模式
3. 零件词汇表，附配方
4. 色调
5. 画家顺序
6. 相机与构图
7. 主题
8. 圆形零件与管道
9. 爆炸视图
10. 屏幕与表面上的物件
11. 着色器

## 1. “精心制作”意味着什么

被批准的图共有五个品质。

1. **一个你能真正造出来的实物。** 读者一眼就能说出它是什么：一个测试台、一个千分表、一个抽屉柜、一个龙门绘图仪、一把锁。它是为主题而发明的，绝不从参考图描摹。
2. **许多小而精确的零件。** 一张好的内联图有 30–60 个实体和 300–500 条路径。一个真正有更多零件的主角可以承载更多：桌面电脑约有 85 个实体，而 Raptor 在 478 个按深度排序的项中约有 270 个实体。每个零件都搁在另一个零件上，与相邻零件对齐，并且各司其职。密度来自结构（支脚、导轨、限位块、衬套、旋钮），而非来自纹理。
3. **体积感。** 零件有真实的厚度，侧面用四种色调着色，顶面更亮，倒角捕捉光线，下方有柔和的晕影。
4. **平静。** 细至 0.6 px 的线条和克制的调色板。一个点亮的强调色告诉眼睛该往哪里看。运动经过缓动，慢到足以跟随。
5. **真实。** 读数显示实时、正确的数字，运动遵循真实模型：具有真实刚度的弹簧、真实的尺寸和阈值。

## 2. 反模式

审阅者拒绝了以下每一项，所以不要这样做：

- **抽象堆叠。** 代表“层”的悬浮板、平板或瓦片太抽象，读不出任何东西。如果你需要层，就把它们做成抽屉、机架中的托盘、导柱上的板，或支架中的卡片。
- **一整块大平板。** 一个只有几条线的大表面太扁平、太乏味。把它拆成底座、滑架和零件；加一道凹槽、几颗螺丝、一把尺子和一块标签板。
- **蓝图式家具。** 引线、带编号的圆圈、尺寸箭头、剖面线、虚线隐藏边和悬浮标签属于图例和说明文字，不属于图本身。
- **停在半空中的线。** 导向扇、在零件前就结束的连接件，或在轮廓拐角前就停止的折痕。每条线都必须落在某个东西上。
- **线框。** 能看见后边缘的透明盒子。
- **到处是颜色。** 除非颜色就是主题，否则保持单色：一个点亮的强调色，加上仅用于颜色通道的真实 RGB。流体默认不是彩色的：视镜中的油、罐中的水和管道中的冷却液都是图自身墨色的淡色调，而 `--no-webgl` 回退绘制与着色器相同的色调。一个盲测台在原本灰色的图中把它的油涂成了饱和的橙色，而它的回退油是灰色的。
- **图中的文字。** 最多允许一个微小的雕刻标签（表面上的 `FaceText`）。其他一切都在卡片角落、图例或说明文字中。
- **照抄参考图。** 从参考图中汲取氛围和质感，然后发明那个物件。
- **原始的指针跳变。** 零件瞬间吸附到光标。一切都要缓动。
- **管道穿过管道。** 在第一版 Raptor 上，物主放大后发现电缆和管线穿过主管道并相互穿过，“这毫无道理”。在 1× 下看起来没问题。每根管道和电缆都避开除自身连接之外的一切，每个交叉都按真实深度顺序绘制，而 `kit/audit.mjs` 证明这两点（第 8 节，“间隙与交叉”）。
- **戛然而止的纹理。** 在同一张特写中，燃烧室上成排的螺栓点纹在表面中间戛然而止，钟形和喉部被着色成阶梯状的平色调块。两者读起来都像坏掉了。物主的原话：如果纹理必须结束，“我们宁愿在末端让它们淡出，而不是瞬间停止”。平滑地为曲面着色，并让纹理结束在真实的边缘上或淡出（第 8 节，“曲面上的着色与纹理”）。
- **突然跳变或被切断的着色器。** 第一版 Raptor 的羽流在其各阶段之间瞬间变色，“仿佛被剪出来一样”，而一道辉光在画布边缘以一条直线结束。每个着色器项都乘上一个包络，并在任何边界之前归零（`webgl.md`，第 7 节）。
- **纹理戛然而止，在平面上也是如此。** 同样的错误在后来一张图的盒子上再次出现，那里没有曲面规则能触及它：一个罐体的焊缝从每个圆角内缩 6 个单位，一条歧管面线内缩 4，一排淡淡的刻度线悬浮在光秃的底板座上、下方没有尺子，冷却肋在两端都距接缝弧线差 2 个单位就停止，而一根软管的编织被画成沿中心线的虚线笔画，沿软管以两条笔直的硬边结束，并在每个分段接缝处断开。盒子上的接缝从边到边绕过可见的拐角（`k.sideSeam`），肋在相同的 `s` 处结束于接缝（`ribsOf(…, { seams: true })`），刻度线从尺条的边缘垂下，而管道上的纹理来自 `tubePieces(…, { rings })`。`node scripts/lines.mjs page.html` 会让每一条不接触任何东西的线端失败（`verify.md`，“线端”）。
- **替代形状。** 为了让审计安静下来，一个构建者在五根重叠杆副本的每一根尖端记录了一个 0.45 厚的圆盘，并为第二根杆画了一个薄片，而它被画成沉入其缸盖中。审计通过了，对两根杆都没说什么，而点亮的杆在移动时其轮廓每 12 个单位就有一个缺口。记录你所绘制之物的真实形状；审计的替代形状检查会让一个记录形状覆盖其绘图不足 60% 的项失败。一根从密封套中滑出的杆就是一根杆（第 3 节，“从密封套中伸出的杆”）。
- **贴在白色上的深色主题辉光。** 物主把浅色定为每个站点图的发布主题，而七个建立在深色底板上的着色器图必须被转换。拿一个深色辉光并把它调暗，得到的是“红墨烟雾”（Raptor 的浅色羽流）。它还得到画成轮廓的火焰、看起来像污渍的火花、鲑鱼色的冷却玻璃、灰色的渐变，以及随着火熄灭而变得更重的空腔。在纸面上，光通过饱和度和更暗的物理底色来读取（第 7 节，以及 `webgl.md`，第 8 节）。
- **靠肉眼修复一张特写。** 调整画家的键或微调一个路点直到某张截图看起来正确，会破坏另一帧或另一个主题。Raptor 手工设定的键在 1× 下看起来正确，却有 323 个深度顺序问题。先测量（第 8 节，“间隙与交叉”），再看。

## 3. 零件词汇表，附配方

`P` 是投影，`k` 是工具包，如 `import * as k from "./iso-kit"`。高度以世界单位计，z 轴向上。按列出的顺序绘制每个配方的实体。

### 带支脚的底板
```js
const PLATE = { x: -128, y: -64, w: 262, d: 128, r: 8 };
halo:  `<path class="iso-halo" d="${k.haloOf(PLATE, -17, 12, P)}"/>`
feet:  for (const [x, y] of byDepth(k.corners(PLATE, 22))) {
         k.cylinder(x, y, 7, -17, 1.6, P, 28)            // flange, tone lo
         k.cylinder(x, y, 5.4, -15.4, 3.4, P, 28)        // foot, tone mid
       }
slab:  k.slabOf(PLATE, -12, 12, P, 8, 1.6)                // tone mid, bevel 1.6
groove: k.planOutline(k.insetPlan(PLATE, 5), 0, P)         // line lo
screws: k.corners(PLATE, 10) → k.ring(x, y, 2.2, 0, P, 16) (line lo) + dots size .5
```
用 `byDepth = (pts) => [...pts].sort((a, b) => k.depthOf([a[0], a[1], 0], P) - k.depthOf([b[0], b[1], 0], P))` 对支脚排序。

### 尺条与标签板
```js
const strip = { x: -110, y: PLATE.y + PLATE.d - 15, w: 230, d: 7, r: 1 };
k.slabOf(strip, 0, 0.6, P, 3)                                   // tone lo, crease none
const t = k.topTicks(strip.x + 4, strip.x + strip.w - 4, 4, 5, strip.y + 0.8, 0.6, [2, 3.6], P);
t.minor → line lo, t.major → line mid
label plate: k.slabOf(plan, 0, 0.8, P, 4) + k.planOutline(k.insetPlan(plan, 1.6), 0.8, P) (faint)
             + three k.lineOnTop rule lines of different lengths (lo, { free: true }) + two screw dots
```
刻度线只存在于尺条或表盘面上，并扎根于其边缘：`topTicks` 让每条刻度线从 `edge` 生长，所以把 `edge` 放在尺条的边缘上（或距其 1 个单位以内），而 `sideTicks` 把它们从 `top` 垂下，所以要传入面的顶边，而不是其下方 2 个单位处。光秃板子上的一排标记是一条两端都悬浮的纹理。标签板的横线代表印刷文字，所以它们是唯一可以停在光秃面上的一处线：传入 `free: true`，让线检查知道这是有意为之。

### 盒子上的接缝、焊缝与装饰线
```js
const seam = k.sideSeam(TANK, 14, P, 6);                         // visible near half of the plan at z = 14, round the corners
k.solidSvg(k.slabOf(TANK, 6, 98, P, 6, 1.2), { tone: "mid", inner: k.lineSvg(seam, { tone: "faint" }) })
```
盒子上的接缝、焊道、箍带或盖线绕过可见的拐角，并恰好结束在垂直的轮廓边上，就像圆柱上的 `sideArc` 一样。传入板自身的 `steps`，让接缝遵循相同的面片，并把它作为 `inner` 传入，使其两端位于轮廓之下。只属于一个面的线（加强筋、面板接缝）贯穿整个面，从边到边：`segment([x, y, z0], [x, y, z1])`，其中 `z0` 和 `z1` 是该面的底部和顶部，绝不是 `z0 + 3`。槽和通风口是闭合轮廓（一个薄圆角平面的 `planOutline`，或在面平面中的相同物），所以它们根本没有端点。绝不要把线从其拐角内缩几个单位：在 4× 下它读起来像一条戛然而止的纹理。

### 从密封套中伸出的杆
一根活塞杆、一个柱塞或一个可见长度随运动而变化的滑块就是一根杆，一次按全长（可见长度加行程）构建，密封套内的部分被裁剪掉：
```js
const ROD = G.rodOut(F, gland, tip, r, P, { stroke });           // F.a points out of the gland
svg = `<clipPath id="rod"><path data-hold d="${ROD.clip}"/></clipPath><g clip-path="url(#rod)">${k.solidSvg(ROD.paths, { lit: true })}</g>`
R.put({ ..., move: G.mul3(F.a, stroke), svg, shapes: [R.solid(A.slide(F, gland, tip, r, { name: "rod" }))] })
live: el.setAttribute("transform", `translate(${dx} ${dy})`); holdStill(el, [dx, dy])   // dx, dy: the item's screen offset this frame
```
`rodOut` 返回从 `gland − stroke` 到 `tip` 的杆路径，以及一个从密封套面到 `tip + stroke` 覆盖它的裁剪。裁剪路径带有 `data-hold`；任何移动该项的东西都用相同的屏幕偏移调用 `holdStill(el, [dx, dy])`，它把裁剪向相反方向移动，所以当杆滑过密封套面时，密封套面保持不动。同一个辅助函数在杆向另一方向移动时把杆缩回密封套（一个负载缸）。在密封套上画一个淡淡的密封环（在密封套面处的 `circleOf`，半径 `r + 0.4`）。用 `A.slide` 记录它，审计通过从密封套面拉伸杆而不是平移它来摆姿势，所以间隙、深度和实体检查在每一帧都看到真实的杆。绝不要用重叠的副本伪造伸出：每个副本的点亮边缘在下一个副本的填充开始处停止，而轮廓在移动时每隔几个单位就有一个缺口。

### 导轨、端部限位块与滑架
```js
rails:  for y of [-20, 15]: k.slabOf({ x: 2, y, w: 122, d: 5, r: 1 }, 0, 3, P, 3, 0.4)   // mid
stops:  k.slabOf({ x, y: -23, w: 5, d: 46, r: 1.2 }, 0, 7, P, 3, 0.5) + 2 screw dots on top
carriage (built at x = 0, moved with translateAlong):
        k.slabOf({ x: -28, y: -24, w: 56, d: 48, r: 3 }, 3, 5, P, 5, 0.6)
        bushings: small slabs at the four corners, taller than the plate, with screw dots
        rivets: a row of 0.4 dots along the front edge
        handle: a small neck slab plus a knurled cylinder on the front face
```

### 立柱、套环、夹块与滚花螺钉
```js
k.cylinder(px, py, 5, z0, 4, P, 28)              // collar
k.cylinder(px, py, 3.2, z0 + 4, h, P, 28)        // post, crease none
k.sideArc(px, py, 3.2, z, P, 16)                 // faint seam rings on the post
k.slabOf({ x: px - 7, y: py - 7, w: 14, d: 14, r: 2.5 }, zc, 12, P, 4, 0.6)   // clamp block
thumb screw: a 2.8 × 3 neck slab from the clamp face + k.cylinder(r 2.6, h 6) + k.knurl(..., 18)
```

### 带指针的滚花旋钮
```js
k.cylinder(cx, cy, 6, z, 4, P, 32)                          // mid
k.knurl(cx, cy, 6, z, z + 4, 36, P)                         // line lo
k.lineOnTop([cx - 3, cy], [cx + 4.6, cy], z + 4, P)         // pointer, line hi
```

### 绕杆的螺旋弹簧（为画家顺序拆分）
```js
const c = k.coil(xFrom, xTo, [rodY, rodZ], 4.2, 6, "x", P);
c.back  → line lo, drawn BEFORE the rod
rod     → k.slabOf({ x, y: rodY - 2, w, d: 4, r: 1.8 }, rodZ - 2, 4, P, 3)
c.front → line mid, drawn AFTER the rod
```
当它压缩时每帧重建螺旋，通过改变 `xFrom`/`xTo`。它很廉价。

### 千分表（面朝上）
```js
k.cylinder(x, y, R, z, 10, P, 64, 0.8)                       // puck, tone hi, crease lo
k.knurl(x, y, R, top - 3.4, top, 84, P, 0.3)                 // knurled bezel band, lo
k.sideArc(x, y, R, top - 3.4, P, 40)                         // band seam, faint
k.ring(x, y, R - 2.2, top, P, 64)                            // bezel ring, lo
const t = k.radialTicks(x, y, R - 3, 50, 5, top, [1.3, 2.8], P)   // minor lo, major mid
crown: two small slabs on the +x side; lug: a slab joining the arm
needle: k
```