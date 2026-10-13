# 安全

## 范围说明

Anatomy 是一组指令、一个绘图工具包以及本地 Node 脚本。启动 Chrome 的脚本（`capture.mjs`、`drive.mjs`、`lines.mjs`、`turn-check.mjs`）以无头模式运行，使用一次性配置文件并带有 `--allow-file-access-from-files` 参数，而 `drive.mjs` 会在页面中执行你传入的 JavaScript。请仅将它们指向你信任的页面。

安全问题是本仓库中任何可能让页面、提示词或文件以用户未要求的方式运行代码、读取文件或访问网络的内容：包括工具包、脚本、构建的示例页面，或 `SKILL.md` 和 `references/` 中可能导致 Claude 做出不安全行为的指令。

## 支持的版本

仅支持 `main` 分支上的最新提交。

## 报告漏洞

请不要为安全问题创建公开 Issue。请通过 GitHub 的[私有漏洞报告](https://github.com/wheresryan22/anatomy/security/advisories/new)功能私下报告本仓库的问题。如果该链接无法使用，请直接在 X 上私信 [@wheresryan22](https://x.com/wheresryan22) 请求一个私密渠道，不要附带详细信息。

请包含你发现的内容、如何复现，以及它可能让他人做什么。