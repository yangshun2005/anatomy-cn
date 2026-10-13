

> **[中文化] anatomy**
>
> 此项目是 [wheresryan22/anatomy](https://github.com/wheresryan22/anatomy) 的中文翻译版本。
> - 原项目 Stars: 7459
> - 主语言: JavaScript
> - 许可证: MIT
> - 翻译日期: 2026-10-13
> - 原始 README: [README_en.md](README_en.md)
>
> 说明：下方 README 正文为**原项目文档的中文翻译**，力求忠于原文；本页顶部的「中文快速上手」为**面向中文用户的补充导读**（非原文内容），目的是让你不用读完整篇 README 就能先把项目跑起来。
>
> 如有翻译不准确之处，欢迎提 Issue 或 PR。

---


## 🚀 中文快速上手（补充导读，非原文）

### 这是什么 / 解决什么问题

Anatomy 是一个 Claude 技能（skill），用来把一个抽象的技术概念「造」成一个真实、可运转的实体图形来解释它。比如把「缓存命中才跳过工作」画成一把只有所有弹子都对上才转动的弹子锁。它生成的是带标题、提示和实时读数卡片的多零件等距（isometric）图形，支持 SVG、SVG + WebGL 和 3D 三种形式。它也能在 OpenCode 和 Codex 中运行。

### 快速上手（最短路径）

1. 安装技能（全局安装到 Claude Code）：
   ```sh
   npx skills add wheresryan22/anatomy -g -a claude-code
   ```
2. 若使用 OpenCode 或 Codex，把 agent 换成 `-a opencode` 或 `-a codex`。
3. 在 Claude Code 中输入命令：
   ```text
   /anatomy explain how a token bucket rate limiter works, interactive
   ```
4. OpenCode 使用同样的 `/anatomy` 命令；在 Codex 中输入 `$anatomy`。
5. 无需 API Key，也无需 `npm install`，该工具只使用 Node 内置模块。

### 环境要求

- 操作系统：原文未说明
- 硬件：原文未说明
- 依赖：无依赖（dependencies: 0），只使用 Node 内置模块
- 版本要求：Node ≥ 20.10
- 网络：原文未说明是否需要更换镜像源或加速下载

### 常见坑与注意事项

- 需要 Node ≥ 20.10，版本过低可能无法运行。
- 3D 模式目前为 beta 状态。
- 不同 agent 的调用方式不同：Claude Code 和 OpenCode 用 `/anatomy`，Codex 用 `$anatomy`。
- 原文未说明是否需要 API Key 或是否需下载大文件。

---



---

<h1 align="center">Anatomy</h1>

<p align="center"><b>一个 Claude 技能，通过将想法构建为真实、可运作的物体来解释它。</b><br>也可在 OpenCode 和 Codex 中运行。<br>精心制作的交互式等距图形，包含许多小而精确的零件，使用 SVG、WebGL 和 3D 呈现。</p>

<p align="center">
  <a href="https://code.claude.com/docs/en/skills"><img src="https://img.shields.io/badge/Claude_Code-skill-1b1b19?style=flat-square&logo=claude&logoColor=white" alt="Claude Code skill"></a>
  <a href="https://agentskills.io"><img src="https://img.shields.io/badge/Agent_Skills-compatible-1b1b19?style=flat-square" alt="Agent Skills compatible"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-1b1b19?style=flat-square" alt="MIT license"></a>
  <a href="#requirements"><img src="https://img.shields.io/badge/node-%E2%89%A5%2020.10-1b1b19?style=flat-square&logo=nodedotjs&logoColor=white" alt="Node 20.10 or later"></a>
  <a href="#requirements"><img src="https://img.shields.io/badge/dependencies-0-1b1b19?style=flat-square" alt="No dependencies"></a>
  <a href="https://github.com/wheresryan22/anatomy/actions/workflows/ci.yml"><img src="https://github.com/wheresryan22/anatomy/actions/workflows/ci.yml/badge.svg" alt="CI"></a>
</p>

<p align="center">
  <a href="https://skills.wheresryan.sh/anatomy"><img src="https://img.shields.io/badge/Website-skills.wheresryan.sh%2Fanatomy-1b1b19?style=for-the-badge" alt="Website: skills.wheresryan.sh/anatomy"></a>
  &nbsp;
  <a href="https://github.com/wheresryan22/anatomy/archive/refs/heads/main.zip"><img src="https://img.shields.io/badge/Download-ZIP-1b1b19?style=for-the-badge&logo=github&logoColor=white" alt="Download ZIP"></a>
  &nbsp;
  <a href="#install"><img src="https://img.shields.io/badge/Install-npx_skills_add-d7d7d0?style=for-the-badge&labelColor=1b1b19&logo=npm&logoColor=white" alt="Install with npx"></a>
</p>

<p align="center">
  <a href="#quick-start">快速开始</a> ·
  <a href="#gallery">图库</a> ·
  <a href="#what-it-makes">它能制作什么</a> ·
  <a href="#install">安装</a> ·
  <a href="#usage">用法</a> ·
  <a href="#examples">示例</a> ·
  <a href="#how-it-works">工作原理</a> ·
  <a href="#limits">限制</a>
</p>

<p align="center">
  <img src=".github/assets/raptor-engine-firing.webp" width="640" alt="一幅等距绘图，展示一台悬挂在测试台架上点火的 Raptor 2 火箭发动机：由 WebGL 着色器绘制的红色尾焰倾泻到台架的底板上。卡片上写着 Fig 1, Raptor 2 test stand, Pc 300 bar, 230 tf。">
</p>

<p align="center"><sub><code>examples/raptor-engine</code>，浅色主题，点火状态。发动机和台架是 SVG；尾焰是同一等距相机中的 WebGL 着色器。</sub></p>

---

当你向 Claude 索要一幅解释技术概念的图形时，Anatomy 会让它发明一个物理事物，而这个事物的运作方式*就是*那个概念。一个在没有任何变化时跳过工作的缓存键，变成了一把弹子锁，只有当所有弹子都匹配时才会转动。一个服务于多个画布的 GPU 上下文，变成了一台龙门绘图仪，其单个打印头会依次访问 16 个孔位。

然后 Claude 用数十个小而坚固、带阴影的零件构建出那个事物，把它放进一个带标题、提示和实时读数的带框卡片中，让它在指针和键盘下平稳地运动，并通过放大每个接合处来检查自己的工作。图形显示的每一个数字都来自驱动其零件运动的同一个微小模型。

## 快速开始

```sh
npx skills add wheresryan22/anatomy -g -a claude-code
```

然后，在 Claude Code 中：

```text
/anatomy explain how a token bucket rate limiter works, interactive
```

对于 OpenCode 或 Codex，替换 agent：`-a opencode` 或 `-a codex`。OpenCode 使用相同的 `/anatomy` 命令；在 Codex 中，输入 `$anatomy`。

就是这样。无需 API 密钥，也无需 `npm install`：该工具包仅使用 Node 的内置模块。

## 图库

<table>
  <tr>
    <td width="50%"><img src=".github/assets/raptor-engine-apart.webp" alt="Raptor 2 发动机在测试台架中被拆解：其组件、管道和线束彼此分离地悬挂在台架的立柱之间。"></td>
    <td width="50%"><img src=".github/assets/ripple-tank.webp" alt="一个学校用的水波槽，架在四条高腿上，上方有一盏装在支臂上的灯。振动杆上的两个激振器产生干涉波，其焦散投影到水槽下方的纸屏上。"></td>
  </tr>
  <tr>
    <td><sub><b>raptor-engine</b>：悬停可将其拆解为 11 个组件。审计证明没有任何管道穿过任何物体，无论是静止状态、拆解状态还是其间的每一帧。</sub></td>
    <td><sub><b>ripple-tank</b>：SVG 装置覆盖在两个 WebGL 图层之上（纸上的灯光和焦散、水中的反射），由色散和光学模型驱动。</sub></td>
  </tr>
  <tr>
    <td><img src=".github/assets/turning-dial.webp" alt="一个凸起门板上的密码转盘。一只小巧的金色和铬色机器人手放在转盘上随之转动；一个指尖下发出绿光。"></td>
    <td><img src=".github/assets/dial-indicator.webp" alt="一个安装在支架上的千分表，位于带有刻度条的底板之上。其测杆搁在滑动托架上的三个量块之一上。"></td>
  </tr>
  <tr>
    <td><sub><b>turning-dial</b>：3D 模式（测试版）。当你拖动转盘时，约 90 个零件每帧都会重新投影和重新着色。</sub></td>
    <td><sub><b>dial-indicator</b>：最小的示例，约 300 行。悬停可滑动托架；指针会读出每个量块。</sub></td>
  </tr>
</table>

每张截图都是 [`examples/`](examples) 中某个页面的浅色主题，使用该技能自带的 [`scripts/capture.mjs`](scripts/capture.mjs) 和 [`scripts/drive.mjs`](scripts/drive.mjs) 捕获。构建好的页面已提交：打开 `examples/` 中的任意 `.html` 即可试用。

## 它能制作什么

三种图形。三者共享同一个工具包、同一个相机和同一套工艺规则。

| 类型 | 它是什么 | 示例 |
|---|---|---|
| **SVG** | 核心。具有四色调阴影的实心等距零件，从后到前绘制，放在带实时读数和缓动运动的卡片中。它以独立 HTML 页面、内嵌样式的单独 `.svg` 文件，或 React/Next.js 组件的形式发布。 | `dial-indicator`、`desk-computer`、`arcade-cabinet`、`test-rig` |
| **SVG + WebGL** | 用于线条无法绘制的内容：火焰、尾焰、水、焦散、灯光、辉光、蒸汽。着色器在后层 SVG 和前层 SVG 之间的画布中运行，使用图形自身的等距相机，因此它位于绘图内部。没有 WebGL 时 SVG 仍然可读。 | `raptor-engine`、`ripple-tank` |
| **3D** <sup>测试版</sup> | 在真正的 3D 中转动的零件：旋转的转盘或转台、弯曲的手指、断裂并掉落的部件。运动部件每帧都由工具包自身的规则重新投影和重新着色；图形的其余部分保持静态艺术。 | `turning-dial` |

每个图形都有浅色或深色主题。浅色是该技能首先设计的主题，着色器也包括在内。

## 安装

Anatomy 是一个标准的 [Agent Skill](https://agentskills.io)：一个根目录下带有 `SKILL.md` 的文件夹。本仓库就是那个文件夹。选择一种安装方式：

| | 命令 | 安装到 |
|---|---|---|
| **skills CLI**，为你自己（Claude Code） | `npx skills add wheresryan22/anatomy -g -a claude-code` | `~/.claude/skills/anatomy` |
| **skills CLI**，为你自己（Antigravity） | `npx skills add wheresryan22/anatomy -g -a antigravity` | `~/.gemini/antigravity/skills/anatomy` |
| **skills CLI**，为你自己（OpenCode） | `npx skills add wheresryan22/anatomy -g -a opencode` | `~/.config/opencode/skills/anatomy` |
| **skills CLI**，为你自己（Codex） | `npx skills add wheresryan22/anatomy -g -a codex` | `~/.agents/skills/anatomy` |
| **skills CLI**，为单个项目 | `npx skills add wheresryan22/anatomy` | Claude Code 为 `.claude/skills/anatomy`，Antigravity、OpenCode 和 Codex 为 `.agents/skills/anatomy` |
| **git**，为你自己（Claude Code） | `git clone https://github.com/wheresryan22/anatomy.git ~/.claude/skills/anatomy` | `~/.claude/skills/anatomy` |
| **git**，为你自己（Antigravity） | `git clone https://github.com/wheresryan22/anatomy.git ~/.gemini/antigravity/skills/anatomy` | `~/.gemini/antigravity/skills/anatomy` |
| **git**，为你自己（OpenCode） | `git clone https://github.com/wheresryan22/anatomy.git ~/.config/opencode/skills/anatomy` | `~/.config/opencode/skills/anatomy` |
| **git**，为你自己（Codex） | `git clone https://github.com/wheresryan22/anatomy.git ~/.agents/skills/anatomy` | `~/.agents/skills/anatomy` |
| **git**，为单个项目 | `git clone https://github.com/wheresryan22/anatomy.git .claude/skills/anatomy` | `.claude/skills/anatomy`（Claude Code 和 OpenCode）或 `.agents/skills/anatomy`（Antigravity、OpenCode 和 Codex） |
| **下载**，无需 git | 见下文 | 上述任意文件夹 |

<details>
<summary><b>使用 skills CLI</b></summary>

<br>

[`skills`](https://github.com/vercel-labs/skills) CLI 会找到本仓库根目录下的 `SKILL.md`，并将其安装为 `anatomy`。默认情况下，它安装到当前项目，并询问要设置哪些 agent。`-g` 为你自己安装，`-a claude-code` 仅设置 Claude Code。使用 `-a antigravity`、`-a opencode` 或 `-a codex` 仅设置 Antigravity、OpenCode 或 Codex。

CLI 会复制整个仓库，包括示例（约 10 MB）。它需要较新的 Node.js：CLI 1.7.1 版本要求 22.20 或更高。

</details>

<details>
<summary><b>使用 git</b></summary>

<br>

将仓库克隆到某个技能文件夹中，使 `SKILL.md` 最终位于 `<技能文件夹>/anatomy/SKILL.md`。对于 Claude Code，使用 `~/.claude/skills/anatomy` 或 `.claude/skills/anatomy`。对于 Antigravity，使用 `~/.gemini/antigravity/skills/anatomy` 或 `.agents/skills/anatomy`。对于 OpenCode，使用 `~/.config/opencode/skills/anatomy` 或 `.opencode/skills/anatomy`；OpenCode 也会读取 Claude Code 和 `.agents` 文件夹。对于 Codex，使用 `~/.agents/skills/anatomy` 或 `.agents/skills/anatomy`。对于项目安装，请提交该文件夹，这样你的团队也能获得它。以后要更新，在该文件夹内运行 `git pull`。

</details>

<details>
<summary><b>下载，无需 git</b></summary>

<br>

[下载 ZIP](https://github.com/wheresryan22/anatomy/archive/refs/heads/main.zip)，解压，将文件夹从 `anatomy-main` 重命名为 `anatomy`，然后将其移动到 `~/.claude/skills/`（Claude Code）、`~/.gemini/antigravity/skills/`（Antigravity）、`~/.config/opencode/skills/`（OpenCode）或 `~/.agents/skills/`（Codex）。

或者在 macOS 或 Linux 上用一行命令：

```sh
mkdir -p ~/.claude/skills/anatomy && curl -L https://github.com/wheresryan22/anatomy/archive/refs/heads/main.tar.gz | tar -xz --strip-components=1 -C ~/.claude/skills/anatomy
```

对于 Antigravity、OpenCode 或 Codex，将相同的文件放入 `~/.gemini/antigravity/skills/anatomy`、`~/.config/opencode/skills/anatomy` 或 `~/.agents/skills/anatomy`：

```sh
mkdir -p ~/.gemini/antigravity/skills/anatomy && curl -L https://github.com/wheresryan22/anatomy/archive/refs/heads/main.tar.gz | tar -xz --strip-components=1 -C ~/.gemini/antigravity/skills/anatomy
```

```sh
mkdir -p ~/.config/opencode/skills/anatomy && curl -L https://github.com/wheresryan22/anatomy/archive/refs/heads/main.tar.gz | tar -xz --strip-components=1 -C ~/.config/opencode/skills/anatomy
```

</details>

Claude Code 会在运行中的会话里识别新技能。如果会话启动时 `skills` 文件夹本身不存在，请运行 `/reload-skills`。Antigravity 和 OpenCode 会在聊天中将其列为 `/anatomy`；如果没有显示，请重启。Codex 会在 `/skills` 下列出它，并用 `$anatomy` 运行；如果没有显示，请重启 Codex。

<details>
<summary><b>Codex：如果浏览器检查被阻止</b></summary>

<br>

该技能通过启动无头 Chrome 来检查自己的图形，这会写入一个临时配置文件并通过本地端口与其通信。Codex 在沙箱中运行命令，而该沙箱可能会阻止 Chrome。图形仍会构建；只是其检查会失败。按优先级排序：

1. 当 Codex 请求在沙箱外运行检查时，为该命令批准它。这需要默认的 `approval_policy = "on-request"`。
2. 或者在 `~/.codex/config.toml` 中一次性允许它：

   ```toml
   approval_policy = "on-request"
   sandbox_mode = "workspace-write"

   [sandbox_workspace_write]
   network_access = true
   writable_roots = ["/tmp"]
   ```

   较新的 Codex 版本可能对这些设置使用不同的名称；请查阅你所用版本的配置参考。
3. 或者在普通终端中自行运行检查，例如 `node scripts/capture.mjs page.html --out shots/page.png`。

以 `--sandbox danger-full-access` 启动 Codex 也可以，但它会解除该会话的所有限制，因此请仅用于你信任的项目。

</details>

## 用法

输入 `/anatomy`（在 Codex 中为 `$anatomy`），并说明图形应解释什么：

```text
/anatomy explain how a token bucket rate limiter works, interactive
```

你不必指名该技能。当你要求等距插图、文档、博客文章或落地页的解释性图形，或交互式讲解器时，Claude 也会加载它。Antigravity、OpenCode 和 Codex 也是如此。

<details>
<summary><b>更多提示词</b>，来自该技能的 <a href="evals/evals.json">evals</a></summary>

<br>

```text
/anatomy a figure for our Next.js docs on database connection pooling: a pool of 10 connections, requests borrow one and give it back, and when all 10 are busy new requests wait in a queue. A React component plus a static HTML preview, dark theme.
```

```text
/anatomy an isometric SVG explaining how a CDN edge cache works, cache hit vs miss, TTL of 60 s, for a dark landing page. Just the SVG file and a PNG preview.
```

```text
/anatomy a Bun
```