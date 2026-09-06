---
name: xiaohongshu-image-card
description: Use when generating an image card / 配图 for a xiaohongshu (or other) content note — loads the gpt-image-2-style-library skill for a production prompt, then generates the image **only via ChatGPT 网页版 true image-gen** (hard rule: no Codex, no HTML/render, no other tool), and saves/embeds it into the note via the Obsidian attachment library. Requires Chrome「允许 Apple 事件中的 JavaScript」.
---

# 小红书图文（图卡）生成

为一篇内容笔记生成竖版 3:4 图卡/配图，并存进对应笔记。整条链路：**style-library 出提示词 → ChatGPT 网页版真生图（唯一路径）→ 存附件库 → 笔记嵌入**。

## 硬规则（用户 2026-09-06 定稿）

**生图只用 ChatGPT 网页版（chatgpt.com）。禁止用 Codex 生图、禁止 HTML/CSS 渲染生图、禁止任何其它生图工具。** 不要降级到 Codex 或其它方式，即使更省事也不行。

## 触发

用户说「配个图文 / 生成图卡 / 配图 / 做张图」，且对象是内容笔记（小红书、公众号等）。

## 前置 skill（必须遵守）

- `gpt-image-2-style-library`：**必调**，用它的模板库产出 GPT-Image2 提示词（第一步）。
- `xiaohongshu-city-research`：数据可核验规范——关键数字要有来源，拿不准标「待核实」，不编造。
- `content-output-naming`：笔记命名 / 存放位置 / 状态机。

## 流程

### 1. 明确目标笔记

从对应 note 读 frontmatter + 正文，提取：标题、核心数据（含来源）、标签、平台、系列。

### 2. 用 style-library 出提示词

加载 `gpt-image-2-style-library`，按其六块组装提示词（主体任务 / 构图版式 / 视觉风格材质 / 文字标签 / 比例输出 / 约束负面）。

模板选择：**账号内容以数据为主 → 默认走 `infographic-engine`（信息图引擎）数据呈现图**：深绿品牌 `#2E7D5B` + 红橙紫蓝青绿阶梯多彩数据图（条形/饼图/地图/排名卡/表格），多模块：大标题+数据源条+TOP榜/图表+数据分析+一句话总结+页脚署名，竖版3:4——**直观看数据，不是场景插画**。只有纯情感/生活向（非数据）才用 `scene-storytelling` + 暖色生活感。中文语境用中文写提示词。**生成工具只有一种：ChatGPT 网页版。**

### 3. 生成真图（ChatGPT 网页版 · 唯一路径）

前提：浏览器已登录 ChatGPT + Chrome 已开「查看→开发者→允许 Apple 事件中的 JavaScript」。

方法（AppleScript + Chrome JS，已跑通）：

1. 定位 `chatgpt.com` 页签，**先从所有 chatgpt.com 页签里挑一个「干净」的会话**（避免和历史生成图混在一起；必要时点「新对话」开新会话，以免取到旧的生成图）。**先 `activate` Chrome**（`execute javascript` 需要窗口前台）。用 `document.execCommand('insertText')` 把第 2 步提示词注入 `#prompt-textarea`（ProseMirror）；注入前先 `selectNodeContents(el)` + `execCommand('delete')` 清空旧内容。发送按钮 `button[data-testid="send-button"]` 只在有内容时才渲染——先注入，再（`setTimeout ~500ms` 后）点它。
2. 等 20–60s；生图完成后，生成图是 `src` 含 `backend-api/estuary/content?id=file_...` 的 `<img>`。**注意**：会话里可能有多张历史生成图，**要取「本次」那张**——用会话级标识或「最后一条 assistant 消息里的图」定位，避免抓到旧图；缩略图 `naturalWidth` 可能为 0（未加载完），`naturalWidth>400` 的才是成品（约 1086×1448）。
3. **取图（关键）**：图片在 chatgpt.com 同域，用 **canvas `drawImage` → `toDataURL('image/png')`** 拿 base64。若缩略图 `naturalWidth===0`，**新建独立 `Image`、`src` 指向该 estuary URL**，等 `onload` 后再 `drawImage` 原图（`crossOrigin='anonymous'`）。

坑（务必避开）：
- **别用「下载」**：程序化 `a.click()` 触发下载会被拦（需用户手势）。
- **别用跨域 fetch**：`fetch('http://127.0.0.1:...')` 被 chatgpt CSP 拦（`Failed to fetch`）。用 canvas→base64 最稳。
- **别用 `[data-message-author-role="assistant"] img` 只查最后一条**——生图可能不在该选择器/会话虚拟化导致来源混乱；**务必按「本次生成的会话/图」定位**，否则会取到旧图（如之前误取到一张武汉租金地图）。
- 长中文提示词转义易错 → 先 Base64，页面内 `decodeURIComponent(escape(atob(b64)))` 解码再插入；不要在 bash 里用非引用 heredoc 传含 `\` 的 JS，用 `<<'PY'`（引号 heredoc）+ 环境变量传 base64。
- **10MB 级 base64 别一次性用 `osascript return`**（会丢/限长）。落盘：页面把 `toDataURL` 存到 `window.__X.data`，Python 循环调 `osascript` 读 `window.__X.data.substr(off,n)` 分片（每片长度 4 的倍数），跳过 22 字符前缀 `data:image/png;base64,`，逐片 `base64.b64decode` 追加写文件。

### 4. 存附件库 + 嵌入

- 把 PNG **存到 Obsidian 附件库 `8️⃣ Attachment/image/`**（不要放进 `media` 内容文件夹——图片是附件，不进内容库）。命名：`<note文件名>-配图.png`，场景版 `-配图-场景版.png`，风格库版 `-配图-风格库版.png`。
- 在对应 note 末尾追加/更新「封面 / 配图」段，用 `![[<图名>.png]]` 嵌入（Obsidian 按全库唯一文件名解析，存附件库也能显示）；多版本可并列。
- **emoji 路径（`3️⃣ output` / `8️⃣ Attachment`）在 shell 里编码不稳** → 一律用 Python 处理复制/写 note，不要手打 emoji 路径。

### 5. 呈现给用户

生成/导出图片后，用 `vision_present` 展示，并说明各版本差异供用户选择。

## 硬性要求（用户审美 + 内容规范）

- 风格（看内容类型）：城市选择/城市研究/数据 → **数据呈现图**（深绿 `#2E7D5B` 品牌 + 多彩数据条形/表格/饼图/地图/排名卡，多模块），默认 `infographic-engine`；只有纯情感/非数据生活向才用 `scene-storytelling` 暖色生活感。少 emoji、去模板感。
- **图内不要放 `#` 标签**：`#话题标签` 属于**正文**，不要放进图片里；图片只放标题/副题/数据/来源。
- 数据：精确数字需可核验来源；拿不准标「待核实」，不编造；正文保留数据口径 / 来源 / 局限。
- 禁止：用图形面积制造比例误导、把缺失值画成 0、用「最新」掩盖旧数据、标题文字错拼。
