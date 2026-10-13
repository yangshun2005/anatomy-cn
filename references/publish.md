# 将图表发布到社区画廊

社区画廊是仓库 `wheresryan22/anatomy-community`。人们在那里分享纯 HTML 图表：一个构建脚本，将独立页面写在自身旁边，就像该技能的示例所做的那样，适用于任何主题。React 和 Next.js 图表无法提交。

仅在用户要求分享或发布图表时才发布。未经他们明确同意，绝不发送任何内容。

## 步骤

1. **检查图表是否已完成。** 其页面已构建且为最新，线条检查通过，如果包含管道、线缆或圆形部件则 `--audit` 通过，并且在开启减弱动态效果时保持一个静止姿态。画廊也会检查所有这些内容。
2. **向用户询问只有他们才能提供的信息：**
   - 他们的 GitHub 用户名（作者）；`gh api user --jq .login` 显示 GitHub CLI 当前登录的身份；
   - 一个标题（最多 80 个字符）和一个简短的 kebab-case id；
   - 许可证：MIT（默认）或 CC-BY-4.0；
   - 确认这是他们自己的作品，并且他们乐意分享。
   提示词是制作该图表的请求；复用它，裁剪到 600 个字符。
3. **获取画廊的本地克隆，** 如果他们还没有：`gh repo clone wheresryan22/anatomy-community -- --single-branch`（仅 `main`：previews 分支保存所有图像并且会变得很大）。克隆主仓库即可：如果需要，提交步骤会为他们创建一个 Fork。
4. **展示计划，** 不做任何操作：
   ```sh
   node <skill>/scripts/publish.mjs <figure folder> --into <clone> --id <kebab-id> \
     --title "..." --author <github user> --prompt "..." \
     [--agent "Claude Code"|OpenCode|Codex|Antigravity|Other] --model "..." [--minutes 45] [--tokens 1.2M] \
     [--license MIT|CC-BY-4.0] \
     [--tags a,b] [--audit] [--pages a.html,b.html] --submit --dry-run
   ```
   传入 `--agent` 表示你正在其中运行的 agent，传入 `--model` 表示你所使用的模型（画廊要求提供，例如 `claude-opus-5-5` 或 `gpt-5-codex`）。在 Claude Code 中，制作时间和消耗的 token 会从本次会话的日志中自动读取：从制作该图表的请求到发布它的请求，超过 10 分钟的空闲间隔不计入，缓存读取不计算在内。在其他 agent 中，如果你能看到会话的使用情况，请传入 `--minutes` 和 `--tokens`；否则请省略它们，而不是猜测。向用户复述该计划：它会指明 Fork（如果有）、分支、它添加的文件夹以及 Pull Request 的去向。
5. **仅在用户同意后，** 运行相同的命令，用 `--yes` 代替 `--dry-run`。它仅将图表的源代码复制到 `figures/<author>/<id>/src/`，将其 kit 导入指向克隆的 `.skill/` 链接，记录 kit 指纹，检查构建，将预览保存到 `shots/<author>-<id>/preview.png`，然后仅提交该文件夹，推送，并打开 Pull Request。将它打印的链接提供给用户。
   - 如果它提示某个文件超出了图表文件夹和技能范围，请将该文件所需的内容复制到图表文件夹中，重新构建，然后再次发布。
   - 如果它无法判断哪些页面属于该图表，请传入 `--pages`。
   - 若要在之后发送更新版本，请使用 `--force` 再次运行；它会更新同一个 Pull Request。
6. **不使用 `--submit`** 时，脚本在准备好文件夹后停止，并改为打印 git 和 `gh` 命令；如果用户更愿意自己发送，请使用该方式。

没有终端的用户也可以通过画廊的 **Submit a figure** Issue 表单提交，附上一个包含图表源代码的 GitHub Gist 链接。

## 画廊拒绝的内容

- 网络调用或外部地址；
- `eval` 或 `new Function`；
- frame、cookie，或访问图表周围的页面；
- 除技能、图表自身文件以及 `node:fs`、`node:path`、`node:url`、`node:crypto` 或 `node:process` 之外的构建导入；
- 读取环境变量；
- `src/` 下除源文本之外的文件，或已提交的构建页面；
- 构建时写入除其页面之外的任何内容，或两次运行之间结果不同的构建；
- 在开启减弱动态效果时仍持续运动的图表。

如果用户计划分享图表，请从一开始就按这种方式编写。