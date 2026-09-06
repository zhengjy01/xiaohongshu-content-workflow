# 小红书配图生成 SOP

为小红书内容笔记生成竖版 3:4 首图 / 图卡。**生成只用 ChatGPT 网页版（chatgpt.com）真·生图**（硬规则：禁用 Codex、禁用 HTML/CSS 渲染、禁用任何其它生图工具）。生成图统一存 Obsidian 附件库并嵌入对应笔记。

> 这是一份「给 Agent 照做」的逐步 SOP，与 `skills/xiaohongshu-image-card/SKILL.md`（agent 指令版）配套。文中 `<vault>` 指你的 Obsidian vault 根，路径可换成自己的目录结构。

## 前置条件

- Chrome 已开「查看 → 开发者 → 允许 Apple 事件中的 JavaScript」（AppleScript 驱动 chatgpt.com 所需）。
- ChatGPT 网页已登录（`chatgpt.com`）。
- Obsidian 附件库 `<vault>/8️⃣ Attachment/image/`；目标笔记在 `<vault>/3️⃣ output/media/<平台>/<系列>/`。
- 提示词风格来源：`gpt-image-2-style-library` skill（外部库 `awesome-gpt-image-2`）。

## 步骤

1. **明确目标笔记**：读 frontmatter + 正文，提取标题 / 要点 / 标签；判断是城市选择/数据卡（深绿信息图卡）还是生活/场景（暖色生活感）。
2. **出提示词**：用 `gpt-image-2-style-library` 六块组装（主体任务 / 构图版式 / 视觉风格材质 / 文字标签 / 比例输出 / 约束负面）。账号内容以数据为主时默认走 `infographic-engine`（信息图引擎）数据呈现图；只有纯情感 / 非数据生活向才用 `scene-storytelling`。
3. **开干净会话（关键）**：从所有 `chatgpt.com` 页签里挑一个「干净」的会话（或点「新对话」开新会话），**避免和历史生成图混在一起**，否则会取到旧图。**先 `activate` Chrome**；长中文提示词先 **Base64**，页面内 `decodeURIComponent(escape(atob(b64)))` 解码，`execCommand('insertText')` 注入 `#prompt-textarea`，再等 ~500ms 点 `button[data-testid="send-button"]` 发送。
4. **等生图**：等待 20–60s；生成图是 `src` 含 `backend-api/estuary/content?id=file_...` 且 `naturalWidth>400` 的 `<img>`（约 1086×1448）。**务必定位「本次生成的图」**，不要用「全页最后一张大图」抓（可能抓到旧图）。
5. **取图（关键）**：**不要下载、不要跨域 fetch**；用 canvas `drawImage` → `toDataURL('image/png')` 拿 base64，经 AppleScript 返回，Python 截取 data URL 后 base64 解码落盘。
6. **缩放**：`sips --resampleHeightWidth 1440 1080 <src> --out <dst>`（统一 1080×1440）。
7. **存附件库 + 嵌入**：存到 `<vault>/8️⃣ Attachment/image/<note>-配图-*.png`（用 Python 处理 emoji 路径）；在笔记「封面 / 配图」段用 `![[图名.png]]` 嵌入。
8. **展示**：`vision_present` 呈现；多版本可视情况并列供用户选择。

## 注意 / 踩坑

- **生成图不在** `[data-message-author-role="assistant"] img` 可靠可查（会话虚拟化 / 来源混乱）→ **务必按「本次生成」定位**，否则会取到旧图（曾误取到一张历史武汉租金地图）；建议开新会话避免混淆。
- **别用**程序化下载（`a.click()` 需用户手势被拦）、**别跨域 fetch**（`fetch('http://127.0.0.1:...')` 被 chatgpt CSP 拒，报 `Failed to fetch`）。
- Chrome 未开「允许 Apple 事件中的 JavaScript」时 `execute javascript` 直接报错。
- emoji 路径（`8️⃣ Attachment` / `3️⃣ output`）在 shell 里编码不稳 → 一律 Python 处理。
- 长中文提示词转义易错 → 先 Base64；不要用非引用 heredoc 传含 `\` 的 JS，用 `<<'PY'`（引号 heredoc）+ 环境变量传 base64。
- **10MB 级 base64 别一次性 `osascript return`**（会丢 / 限长）→ 分片读 `window.__X.data.substr(off,n)`（每片长度 4 的倍数，跳过 22 字符前缀）后逐片解码。

## 相关

- Agent 指令版：`skills/xiaohongshu-image-card/SKILL.md`
- 提示词来源：`gpt-image-2-style-library`（外部库 `awesome-gpt-image-2`）
