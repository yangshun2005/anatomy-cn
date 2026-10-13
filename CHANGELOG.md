# 更新日志

Anatomy 的所有重要变更都记录在此。格式遵循 [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)。发布版本遵循[语义化版本](https://semver.org/spec/v2.0.0.html)；首次发布之前的条目按日期记录。

## [未发布]

### 新增

- 新增 Antigravity 支持，与 Claude Code、OpenCode 和 Codex 并列。在 README 和 CONTRIBUTING 中记录了用户级和项目级安装路径，更新了 SKILL.md 兼容性说明，并将 Antigravity 添加到缺陷报告表单中。

### 变更

- Figure 不再向用户报出时间估算。30–90 分钟的估算往往远长于实际构建时间（[#41](https://github.com/wheresryan22/anatomy/issues/41)）。

### 修复

- `capture.mjs` 在操作页面前会等待页面加载完成，即使使用 `--wait 0` 也是如此；当 Chrome 在 60 秒内未响应请求时，会报错退出而不是挂起。
- `drive.mjs` 的 `until` 在页面内部对条件计时，因此即使等待时间很短，已经为真的条件也能通过，在慢速机器上同样如此；而永不 settle 的 promise 仍会超时。
- 在 Windows 上从提权终端启动无头 Chrome：`capture.mjs`、`drive.mjs` 和 `turn-check.mjs` 在该平台上传递 `--do-not-de-elevate`。否则 Chrome 会在没有提权的情况下重新启动自身，脚本启动的进程以退出码 0 退出，检查会以 `Chrome exited before startup: 0` 停止，而重新启动的浏览器仍在运行。
- 在每次写入时解析当前 React readout 目标，使被替换的 readout 和锚点不会将更新遗留在已分离的元素上。
- 当页面没有可调用的 3D turn 控制器时，拒绝 drive 的 `turn` 和 `orbit` 操作，而不是将未变化的图形报告为成功旋转。
- 等待异步 `until` 条件时，不将 promise 本身视为成功，同时强制执行等待截止时间。
- Capture 在每次截图前测量当前图形，包括移动、调整大小和定时缩放裁剪的情况。
- `publish.mjs --submit` 在 gh 2.101 及更高版本中重新创建 Fork，这些版本在指定仓库名时拒绝 `--remote`；在这种情况下 gh 从不添加 remote，因此该标志已被移除（#53）。

## [1.1.0] - 2026-10-12

### 新增

- `scripts/publish.mjs` 和 `references/publish.md`：将完成的免框架图形分享到社区画廊 `wheresryan22/anatomy-community`。该脚本复制图形，将其 kit 导入指向画廊的 `.skill/` 链接，记录 kit 指纹（`scripts/fingerprint.mjs`）以便画廊使用相同 kit 版本重新构建，并仅复制源代码到 `figures/<author>/<id>/` 下。画廊在锁定沙箱中构建页面。`--submit` 在询问后将其作为 Pull Request 发送，必要时创建 Fork。
- `publish.mjs` 记录图形制作耗时和使用的 token 数。在 Claude Code 中，它从会话自身的日志中读取这些数据（`scripts/session.mjs`，通过 `CLAUDE_CODE_SESSION_ID`，包括子代理）：从创建图形的请求到发布它的请求，超过 10 分钟的空闲间隔不计入，缓存读取不计数。计划会在发送任何内容之前显示这两项。`--minutes` 和 `--tokens` 可手动设置，`--no-session` 则省略它们。
- `publish.mjs` 接受 `--agent Antigravity`，与 Claude Code、OpenCode、Codex 和 Other 并列。
- 新增 OpenCode 和 Codex 支持，与 Claude Code 并列。`SKILL.md` 通过 `metadata` 向 OpenCode 暴露 `/anatomy`，`agents/openai.yaml` 为 Codex 提供显示名称，README、CONTRIBUTING 和缺陷报告表单覆盖全部三种代理。

### 修复

- 在 Linux 上启动无头 Chrome：`capture.mjs`、`drive.mjs` 和 `turn-check.mjs` 在该平台上传递 `--no-sandbox`，并在 `~/.cache/ms-playwright` 下查找 Playwright 的 Linux Chromium，使浏览器检查和画廊的验证器能在 Linux 主机上运行。
- `references/publish.md` 使用 `--single-branch` 克隆画廊，使贡献者跳过预览分支及其图片。
- `publish.mjs --submit` 在之前未使用 `--submit` 的运行将图形文件夹留在克隆中时，不再以“已存在”停止；仅当该文件夹已在画廊主分支上时才要求 `--force`。
- 在宽阶段实体和管道采样中保留配置的管道间隙。
- 通过 useInView 观察替换和延迟挂载的 DOM 目标。
- 针对圆角矩形角检查纹理线条附着，包括独立圆角半径、SVG 裁剪和变换后的轮廓。
- 遵循显式的 capture、drive 和 line-check 选择器，并将报告的 readout 限定到所请求的图形。
- 在审计边界中解码有效的 SVG 属性引号、空白和 XML 字符引用。
- 从作者编写的 SVG 边界中排除不可见的空裁剪交集。
- `scripts/drive.mjs` 现在在浏览器求值抛出或拒绝时失败，而不是报告一个干净的成功会话。
- 街机柜现在在启用减弱动态效果时接受硬币，在跳过硬币动画时保留计量表、信用和开机事件。
- 旋转表盘在页面打开时启用减弱动态效果后停止其空闲巡游，并取消待处理的自动重启，同时保留键盘和指针输入。
- `scripts/drive.mjs` 现在在 `until` 条件超时时以状态码 1 退出并停止后续操作。
- `scripts/capture.mjs` 现在为截图创建输出目录，包括 `shots/` 下的嵌套路径。
- 当 `--select` 指定 SVG 时，线端检查现在包含所选元素，而不是仅检查其内部的 SVG。
- `scripts/drive.mjs` 现在在运行会话前拒绝未知操作，因此拼写错误的验证步骤不会报告成功。
- 当 Chrome 在其调试端口就绪前退出时，浏览器检查现在会迅速失败并移除其临时配置文件。
- 线端检查现在使用椭圆边界和变换后的圆形边界，并将空心内部视为空而非实心点。
- 线端检查现在沿 SVG 弧线曲线进行，而不是将每条弧视为直线弦。
- Capture 和 drive 现在在 Chrome 无法加载所请求页面时失败，而不是报告一个干净的浏览器错误页面。
- Orbit 检查在零状态扫描可能通过之前拒绝无效的采样步长和空姿态列表。
- 实体间隙审计在采样间距或单元格大小变化时重建缓存的表面网格。
- 管几何审计使用锥形半径，而不是将每个段替换为平均圆柱体。
- 线验证器仅在匹配的曲线族之后反映平滑 Bezier 控制点。
- Capture 和 drive 发送原生 Enter 激活，使按钮和表单检查执行其默认操作。
- 固定部件和覆盖边界在相对移动之前恢复闭合 SVG 子路径原点。
- 审计射线投射现在在投影方向变化时刷新物体相机系数，使重复构建保持正确的深度顺序。
- 审计轮廓追踪现在检查四分之一跨度偏差，保留中点位于其弦上的拐点解析和平滑轮廓。
- 深度键求解现在允许大型无环图形在超过 400 轮后收敛，并避免从空的违规规则集中选择循环牺牲者。
- 拆分现在接受默认固定轮次并保留显式的棱柱/圆形归属，因此管道配件可以在不与其自身管道产生碰撞的情况下拆分。
- 旋转构建器现在在作者编写的运动静默变为静态或别名同一值之前，拒绝缺失的命名部件/父组和重复的组名。
- 原生保真度检查现在比较切割部件的完整填充轮廓，并遵循 kit 参考中的开放圆形端，保留故意不描边的切割边缘。
- 管保真度检查在顶点数不同时双向检查路径覆盖，拒绝空或截断的实时轮廓，同时保留等效重采样。
- 实体间隙检查绘制范围变化的滑动杆，即使其拉伸几何体与静止障碍物之间没有相对平移。
- React `Solid` 遵循现有的 `paths.seamless` 标记，使平滑色调区域获得与纯 SVG 渲染器相同的接缝处理。
- React `Dots` 保留 kit 的可选逐点不透明度，包括渐隐的螺栓圆和脉动点，而普通点保持不透明。
- 即使为空或在被动绑定之前发出，也写入第一个 readout 值。
- 通过真实仿射边界测量变换后的矩形和圆形 SVG 图元。

## [1.0.0] - 2026-10-10

### 新增

- `examples/arcade-cabinet/`，一台侧板打开的立式街机柜。一枚 25 分硬币沿投币槽滚下，经过投币开关线进入钱箱，开关的脉冲沿其线缆传到电路板，CRT 预热、自检并运行在倾斜显示器平面内绘制的吸引模式；摇杆闭合面板下的微动开关。它支持浅色和深色主题绘制，并通过 `--audit`。
- `examples/spot-plate/`，来自与 `test-rig` 同一站点的第二个 React 图形：六对玻璃水滴在触片扇设置 `merge` 时汇合成一块玻璃。它支持浅色和深色主题绘制。
- 第一个工作流步骤，**开始前先评估规模**：每个请求被评估为 Figure（约 30–90 分钟）、Hero（2–4 小时）或 Epic（6 小时或更多），并首先将估算告知用户。在用户选择 Epic 而非提议的 Hero 版本之前，不会启动 Epic。每种规模都会在第一个小时内有一个可工作的页面，当某个步骤超过其估算份额的两倍时，Claude 会报告。

### 修复

- `examples/test-rig/` 在浅色主题下将其玻璃按钮绘制为深色。`stretch.css` 现在为其自身颜色提供了浅色值。

### 变更

- 工作流步骤重新编号为 1–10。
- 4× 细节扫描随规模缩放：Figure 扫描主体和机制，Hero 或 Epic 扫描整个绘图。密度规则本身仍适用于每个图形。

## 2026-10-08

### 变更

- 该技能现在称为 Anatomy：`SKILL.md` 中为 `name: anatomy`，以 `/anatomy` 调用。在此日期之前，它以 `isometric-objects` 私下开发，没有发布版本。
- CSS token 现在为 `--anatomy-*`（例如 `--anatomy-card`、`--anatomy-face`、`--anatomy-shade-0`）；之前为 `--iso-*`。每个示例都已用新名称重建。
- 图形卡片（`.iso-plate`）为方形且扁平：没有圆角，没有浮动阴影，仅在绘图中间色调中有一条 1 px 的线。
- `SKILL.md` 将高细节作为每个图形的固定规则，将部件列表提高到至少 40 个部件（英雄图形为 60–150 个），并在最终检查中添加 4× 细节扫描。

### 新增

- 公开仓库：包含示例截图的 README、MIT 许可证、贡献指南、行为准则、安全策略、Issue 表单、Pull Request 模板，以及一个 CI 工作流，该工作流会重建每个示例并在已提交页面发生变化时失败。
- 此时该技能包含的内容：`SKILL.md` 中的工作流和工艺规则；kit（SVG 核心、圆形部件、管道、几何审计、WebGL 层、React 组件，以及 beta 版 3D 模式）；八个参考文档；`scripts/` 中的无头 Chrome 检查；六个示例（`dial-indicator`、`desk-computer`、`raptor-engine`、`ripple-tank`、`test-rig`、`turning-dial`）；以及五个评估。