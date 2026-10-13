# 为 Anatomy 做贡献

感谢你的帮助。Anatomy 的评判标准是它的图形在 4 倍放大下的表现，所以本指南的大部分内容都是关于在别人检查你的工作之前先自行检查。

参与即表示你同意遵守[行为准则](CODE_OF_CONDUCT.md)。

## 环境搭建

你需要：

- **Node.js 20.10 或更高版本。** 无需安装任何东西：工具包、构建脚本和检查脚本只使用 Node 的内置模块。
- **Google Chrome 或 Chromium**，用于浏览器检查。脚本会在常见位置查找它；否则将 `CHROME_PATH` 设置为该二进制文件的路径。
- **Claude Code**、Antigravity、OpenCode 或 Codex，用于试用该技能本身。

```sh
git clone https://github.com/wheresryan22/anatomy.git
cd anatomy
```

要在修改的同时将你的工作副本用作技能，请将其链接到你的个人技能文件夹（先将任何已安装的副本移开）：

```sh
ln -s "$PWD" ~/.claude/skills/anatomy
```

对于 Antigravity、OpenCode 或 Codex：

```sh
ln -s "$PWD" ~/.gemini/antigravity/skills/anatomy
```

```sh
ln -s "$PWD" ~/.config/opencode/skills/anatomy
```

```sh
ln -s "$PWD" ~/.agents/skills/anatomy
```

## 构建示例

每个无框架示例都是一个脚本，它会在自身旁边生成独立页面。构建后的页面会被提交到仓库。

```sh
node examples/dial-indicator/build.mjs
node examples/dial-indicator/build.mjs --light
node examples/desk-computer/build.mjs
node examples/desk-computer/build.mjs --light
node examples/arcade-cabinet/build.mjs
node examples/arcade-cabinet/build.mjs --light
node examples/ripple-tank/build.mjs
node examples/ripple-tank/build.mjs --light
node examples/raptor-engine/build.mjs
node examples/raptor-engine/build.mjs --light
node examples/turning-dial/build.mjs
node examples/turning-dial/build.mjs --light
node examples/turning-dial/build.mjs --light --verify
node examples/turning-dial/stress.mjs --light
```

Raptor 的构建每次需要几分钟；其余的只需几秒。`examples/test-rig/` 和 `examples/spot-plate/` 是没有构建脚本的 React 组件。

构建之后，`git status --short` 必须不显示任何内容，除非你确实想要修改某个图形。CI 会运行完全相同的这些命令，然后执行 `git diff --exit-code`。

## 检查

运行与你所修改内容相匹配的检查，并查看它们生成的每一张 PNG：打开它并仔细观察。将截图放在 `shots/` 中，该目录已被忽略。

**浏览器驱动回归测试。** 在修改 `scripts/drive.mjs` 时，检查失败的浏览器求值是否以非零状态退出，以及成功的求值是否仍然正常工作。

```sh
node --test scripts/drive.test.mjs
```

**几何审计。** 用于带有管道、线缆或圆形部件的图形。它必须以 `audit passed` 结束，且每一行失败项都为 0。

```sh
node examples/raptor-engine/build.mjs --audit
```

**线端检查。** 每条接缝、肋条、刻度和指针都必须终止于一条描边、一个点或一个轮廓上。它必须打印 `line ends: 0`；`--out` 会圈出失败项。

```sh
node scripts/lines.mjs examples/dial-indicator/dial-indicator-light.html --out shots/lines-light.png
node scripts/lines.mjs examples/dial-indicator/dial-indicator.html --out shots/lines.png
```

**截图。** 静帧、手机宽度、悬停、按键和 4 倍特写，并开启 WebGL。

```sh
node scripts/capture.mjs examples/ripple-tank/ripple-tank-light.html --out shots/desk.png
node scripts/capture.mjs examples/ripple-tank/ripple-tank-light.html --out shots/phone.png --width 390
node scripts/capture.mjs examples/ripple-tank/ripple-tank-light.html --out shots/joint.png --zoom 200,80,220,170
node scripts/capture.mjs examples/ripple-tank/ripple-tank-light.html --out shots/still.png --reduced
```

**驱动。** 脚本化会话：移动、点击、按键、等待某个阶段，以及放慢速度以捕捉着色器中突变的联系表。`--no-webgl` 检查 SVG 回退方案。

```sh
node scripts/drive.mjs examples/raptor-engine/raptor-engine-light.html '[["wait",1500],["key","i"],["wait",3000],["shot","shots/firing.png"]]'
node scripts/drive.mjs examples/raptor-engine/raptor-engine.html '[["wait",1500],["shot","shots/no-webgl.png"]]' --no-webgl
node scripts/drive.mjs examples/raptor-engine/raptor-engine-light.html --actions ignite.json
```

任何带有 `until` 或 `eval` 的内容都放在动作文件中。这个 `ignite.json` 等待引擎静止，然后以 20 倍慢动作拍摄点火开始的过程：

```json
[
  ["key", "Escape"],
  ["until", "document.querySelector('[data-readout]').textContent.includes('cold')", 15000],
  ["slow", 0.05],
  ["key", "i"],
  ["sheet", "shots/ignite.png", 100, 16],
  ["slow", 1]
]
```

**3D 图形。** 轨道审计很慢（完整运行每个部件预算 30 到 40 秒），所以用 `--quick` 迭代，并在后台运行完整审计。只有完整运行的 `turn audit passed` 才算数。

```sh
node examples/turning-dial/build.mjs --light --audit --quick
node examples/turning-dial/build.mjs --light --audit > audit.txt 2>&1 &
node scripts/turn-check.mjs examples/turning-dial/build.mjs --fidelity
node scripts/turn-check.mjs examples/turning-dial/turning-dial-verify-light.html --order --step 1
node scripts/turn-check.mjs examples/turning-dial/turning-dial-light.html --lines
node scripts/turn-check.mjs examples/turning-dial/turning-dial-light.html --perf --width 390 --throttle 4
node scripts/turn-bench.mjs examples/turning-dial/build.mjs
```

[`references/verify.md`](references/verify.md) 包含所有选项、审查清单和常见故障。[`references/3d.md`](references/3d.md) 涵盖 3D 检查。

## 内部规则

这些规则来自 [`SKILL.md`](SKILL.md) 和 [`references/`](references)。违反其中任何一条的 Pull Request 都会被要求修改。

- **代码中不写注释。** 不是文件头，不是棘手代码行旁的说明，也不是 JSDoc。将解释放在命名和结构中。诸如 `"use client"` 之类的工具指令不是注释，予以保留。
- **工具包的修改是可选的。** 对 `kit/` 或 `scripts/inline-kit.mjs` 的修改会添加一个选项或函数，其默认值为旧行为。每个现有示例都必须重新构建为字节级一致；CI 会检查这一点。如果某个修改旨在改变图形，请在 Pull Request 中说明，并一并提交重新构建的页面。
- **两种主题都可用，且浅色主题随附发布。** 每种颜色都是一个带有浅色值和深色覆盖的令牌，绝不是图形 CSS 或着色器中的字面量。浅色着色器是为纸张设计的（正常的预乘 alpha，绝不使用 `screen`），而不是将深色辉光调暗。用 `--light` 构建并检查两个页面。
- **不允许重叠。** 任何管道或线缆在静止、分离或运动的任何一帧中都不穿过任何东西。每个交叉都按真实深度顺序绘制，由审计证明，绝不凭肉眼调整。在几何中修复互锁，绝不用关键帧偏置，并记录你所绘制内容的真实形状：一个让审计安静下来的替代形状是一个 bug。
- **线条有归宿，纹理有渐变。** 每条线都终止于某物上。一排肋条、螺栓圆、接缝或标尺线终止于真实边缘或渐隐；它绝不会停在表面中间。曲面平滑着色，没有色调阶梯。
- **着色器绝不突变。** 每个 uniform 都遵循带有上升和衰减的包络，相位变化是交叉淡入淡出，每一项在任何边界或画布边缘之前都归零。效果存在于图形自身相机的世界空间中，且 SVG 在没有 WebGL 的情况下仍可读取。
- **每个数字都是真实的。** 读数、运动和说明文字来自同一个模型，使用真实数字。文档遵循同样的规则：绝不写下你未经验证的数字或功能。
- **查看每一张 PNG。** 在请求审查之前，查看两种主题下每个接缝的 4 倍特写。

## 修改技能文本

`SKILL.md` 和 `references/` 是 Claude 遵循的指令，所以保持它们简短、精确，并使用同样平实的语气。当你修改工具包 API 时，在同一个 Pull Request 中更新 [`references/kit.md`](references/kit.md)。当你添加示例时，在 `examples/` 下给它一个文件夹，其中包含一个 `build.mjs`，它会写入 `<name>.html`，并在使用 `--light` 时写入 `<name>-light.html`；将它添加到 `SKILL.md` 中的列表、README 中的表格以及 [`.github/workflows/ci.yml`](.github/workflows/ci.yml) 中的构建列表。

## Pull Request

1. 对于任何比修复更大的改动，先开一个 Issue，以便我们就方案达成一致。
2. Fork 仓库并从 `main` 创建分支。
3. 每个 Pull Request 只做一处修改。
4. 重新构建每个示例，并确认 `git status --short` 只显示你打算修改的内容。
5. 运行适用的检查，并为任何视觉变化附上两种主题下的截图。
6. 在 [`CHANGELOG.md`](CHANGELOG.md) 的 `Unreleased` 下添加一行。
7. 填写 Pull Request 模板。

合并前 CI 必须通过。

## 报告 bug 或请求图形

使用 [Issue 表单](https://github.com/wheresryan22/anatomy/issues/new/choose)。对于生成错误的图形，请包含提示词、主题、图形所在位置（HTML、SVG 或 React），以及故障的截图或特写。