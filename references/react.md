# React 与 Next.js

## 单个图形的文件布局
```
components/iso/                 工具包：iso-kit.ts、iso.css、react/draw.tsx、react/live.tsx
components/figures/rig/
  model.ts       纯模型：真实数值、弹簧、读数文本（无 DOM）
  view.ts        世界布局常量、投影 P（fitProjection）、辅助函数如 at(point)
  geometry.ts    所有静态路径，在模块加载时计算一次
  frame.ts       形状会变化的部件的路径：frameOf(state) -> { glass, coils, needle, ... }
  art.tsx        服务端组件：静态部件作为 JSX 常量，传递给实时组件
  live.tsx       "use client"：refs、循环、指针和键盘、aria、写入读数
  rig.css        作用域内的 token 和样式（例如 .rig { … }）
  figure.tsx     <Plate fig title hint readout keys caption><RigArt /></Plate>
```
`examples/test-rig/` 遵循此布局。它使用 `rig.ts`、`view.ts`、`geometry.ts`、`pose.ts`、`art.tsx`、`live.tsx` 和 `figure.tsx`。

## 服务端/客户端拆分
- 几何是纯 TypeScript，因此它在服务端运行，客户端接收已完成的路径字符串。除非某个运动部件需要逐帧重建，否则不会发送任何几何代码。此时 `frame.ts` 会被 `live.tsx` 导入并发送到客户端。
- `art.tsx` 将静态部件构建为 JSX（`<Solid paths={BASE_ART.slab} tone="mid" />` 等），并将它们作为元素 props（如 `base`、`rail`、`carriage` 和 `dial`）传递给 `"use client"` 实时组件。实时组件按画家顺序放置它们，并将运动部件包裹在 `<g ref>` 中。
- 只有实时组件和 `kit/react/live.tsx` 是客户端代码。`draw.tsx` 是服务端安全的。

## 大页面 key 警告陷阱
在大页面上，服务端载荷很大，React 会将某些元素 props 作为延迟（lazy）块进行流式传输。客户端在兄弟节点中渲染 `{props.dial}`。它无法将 lazy 标记为已检查 key，因此一旦 lazy 解析为没有 key 的元素，React 就会记录 "Each child in a list should have a unique key"，指向你的实时组件或 `IsoFigure`。同一个图形在小型测试页面上不会记录任何内容，因此逐个检查图形会漏掉它。

有两种修复方法，你应该从一开始就同时做到：

1. **为每个元素 prop 的根节点赋予稳定的 key：** `const DIAL = (<g key="dial">…</g>)`，对于 fragment 使用 `<React.Fragment key="sled">`。单个元素上的 key 没有成本。
2. **让 `"use client"` 组件远离服务端构建的 art。** 服务端树中的每个客户端组件元素都会变成 lazy 引用，这也会使其所有祖先变为 lazy。如果 art 的某些部分需要交互性（例如悬停高亮），请将它们设为普通的 `<g data-part="…">`，并在实时组件中通过事件委托处理事件。

你使用 `.map` 构建的任何数组照常需要 key。

## 样式
- 全局或在图形中导入一次 `iso.css`。在包装器（或 `<body>`）上放置 `className="iso"` 以使用 token，并加上 `data-theme="light"` 以使用浅色主题。
- 将每个图形的 token 作用域限定到图形的类（`.rig { --rig-glass-top: … }`），仅在卡片需要时在 `.iso-plate` 内覆盖工具包 token。
- 不要将图形 token 放在 `:root` 上。

## 读数
`useReadout()` 从其 `anchor` 找到最近的 `.iso-plate [data-readout]`，`say(text)` 写入它而无需重新渲染。在服务端渲染静止文本，以便在水合之前就是正确的。

## 水合
在服务端和客户端首次渲染时渲染静止姿态；仅在 effect 中启动运动。渲染时不要读取 `window`。`asksForStillness()` 是安全的，因为它只在 effect 和循环内部调用。