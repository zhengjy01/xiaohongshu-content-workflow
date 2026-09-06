---
name: xiaohongshu-content-pipeline
description: Use to produce a complete xiaohongshu post end-to-end from a confirmed pain-point topic — confirm the pain point → research → 成稿 (xiaohongshu template) → 配图 via ChatGPT 网页版 only (hard rule) → save to Obsidian attachment and embed → present. Applies to city-choice / lifestyle content (Light｜普通人城市选择指南).
---

# 小红书内容生产流水线

从「痛点选题」到「文字稿 + GPT 配图 + 落地笔记」一条龙。当一个痛点选题被确认后，用本 skill 跑完整链。

## 硬规则（不可违背）

- **生图只用 ChatGPT 网页版（chatgpt.com）**：禁用 Codex 生图、HTML/CSS 渲染生图、任何其它生图工具。
- **选题必须痛点驱动、具体可感**：打在特定人群具体痛点上（收入焦虑/房租成本/存钱/通勤/选择困难/婚恋生育/落户门槛），能让人一眼对号入座。宽泛抽象的「元问题/框架类」（如「XX 值不值得去」「城市怎么选」）先拒绝，或落到非常具体的痛点场景。
- **数据诚实**：来源可核验；拿不准标「待核实」；不编造精确数字；保留数据口径/来源/局限。

## 流程

1. **确认痛点**：把选题转成「具体痛点 + 具体人群」的问题（`城市值不值得去` → `一线房租连涨，月薪 8000 撑得住吗`）。太泛就换更尖锐的痛点。
2. **研究**（走 `xiaohongshu-city-research`）：固定口径、核验数据、标证据等级（官方原始/权威转述/估算/待核实）；能写框架就诚实标明，不硬编数字。
3. **成稿**（`xiaohongshu-city-research` 的「小红书成稿」模板）：5 个备选标题（含克制的问句/数据标题）→ 反差/痛点开头 → 2–4 段拆解（具体数字 + 对普通人的意义）→ 数据口径/来源/局限 → 互动问题 + 署名（🧠 Light｜普通人城市选择指南）。
4. **配图**（`xiaohongshu-image-card` + `小红书配图生成-SOP`）：`gpt-image-2-style-library` 出提示词 → **ChatGPT 网页版真生图**（开干净会话避免旧图、定位本次生成、缩略图要新建 Image 加载原图 → canvas→base64）→ 落盘。
5. **存附件库 + 嵌入**：PNG 存 `8️⃣ Attachment/image/`；笔记末尾「封面 / 配图」段用 `![[图名.png]]` 嵌入；emoji 路径用 Python 处理。
6. **呈现**：`vision_present` 展示成稿 + 配图。

## 分工（复用已有 skill，不重复）

- 研究 / 成稿模板 / 素材挖掘 → `xiaohongshu-city-research`
- 配图生成链路（含 GPT web 取图坑） → `xiaohongshu-image-card` + `2️⃣ AI/Sop/小红书配图生成-SOP.md`
- 命名 / 存放 / 状态机 → `content-output-naming`
