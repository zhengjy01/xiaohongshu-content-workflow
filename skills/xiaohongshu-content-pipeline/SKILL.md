---
name: xiaohongshu-content-pipeline
description: Use to produce a complete xiaohongshu post end-to-end from a confirmed pain-point topic — confirm the pain point → research → 成稿 (xiaohongshu template) → 配图 via ChatGPT 网页版 only (hard rule) → save to Obsidian attachment and embed → present. Applies to city-choice / lifestyle content (Light｜普通人城市选择指南).
metadata:
  group: 内容创作
---

# 小红书内容生产流水线

从「痛点选题」到「文字稿 + GPT 配图 + 落地笔记」一条龙。当一个痛点选题被确认后，用本 skill 跑完整链。

> 「一条龙」指**环节齐全**，不是「无人值守跑到底」——流程被三道人工闸门切开，见下。

## 硬规则（不可违背）

- **生图只用 ChatGPT 网页版（chatgpt.com）**：禁用 Codex 生图、HTML/CSS 渲染生图、任何其它生图工具。
- **选题必须痛点驱动、具体可感**：打在特定人群具体痛点上（收入焦虑/房租成本/存钱/通勤/选择困难/婚恋生育/落户门槛），能让人一眼对号入座。宽泛抽象的「元问题/框架类」（如「XX 值不值得去」「城市怎么选」）先拒绝，或落到非常具体的痛点场景。
- **数据诚实**：来源可核验；拿不准标「待核实」；不编造精确数字；保留数据口径/来源/局限。

## 三个审核闸门（人工放行，禁止自动跨越）

整条流水线被三道闸门切开；**每一道都必须停下等用户放行，agent 不得自行往下走**（用户 2026-09-22 定稿）：

| # | 闸门 | 停在哪一步 | 放行条件 |
| --- | --- | --- | --- |
| 1 | **选题审核** | 每日选题雷达产出后 | 用户从雷达里挑中一条（置 `status: 已选题`）才允许进入研究 |
| 2 | **生图前审核** | 出完「生图计划」（提示词全文 + 计划张数 + 每张用途 + 尺寸 + 模板）后 | 用户确认（或改过提示词）之后才允许打开 ChatGPT 网页版生图 |
| 3 | **发布前审核** | 成稿 + 全部配图齐备后 | 用户决定 **发布**（填链接）或 **打回**（写原因） |

- **打回在闸门 2、闸门 3 都可用**：生图前说「提示词重写」也算打回。打回必须带原因，落到笔记 frontmatter 的 `review_note`；agent 读到原因后修订，修完回到被卡住的那个闸门重新送审。
- **绝不自动发布**：小红书没有开放 API，平台操作一律人工完成，agent 只负责内容与状态。
- 闸门状态取值与回写规则见 `content-output-naming` 的状态机；手机端 Notion 是三道闸门的审核载体（设计见 Mnemon 文档《小红书 Obsidian → Notion 同步方案（设计规格）》）。

## 流程

1. **确认痛点**：把选题转成「具体痛点 + 具体人群」的问题（`城市值不值得去` → `一线房租连涨，月薪 8000 撑得住吗`）。太泛就换更尖锐的痛点。
   → **⛔ 闸门 1**：若选题来自每日雷达，停下等用户挑选题；不得自行选题往下走。
2. **研究**（走 `xiaohongshu-city-research`）：固定口径、核验数据、标证据等级（官方原始/权威转述/估算/待核实）；能写框架就诚实标明，不硬编数字。
3. **成稿**（`xiaohongshu-city-research` 的「小红书成稿」模板）：5 个备选标题（含克制的问句/数据标题）→ 反差/痛点开头 → 2–4 段拆解（具体数字 + 对普通人的意义）→ 数据口径/来源/局限 → 互动问题 + 署名（🧠 Light｜普通人城市选择指南）。
4. **出「生图计划」**：走 `gpt-image-2-style-library` 定模板（数据内容默认 `infographic-engine`），写出**提示词全文 + 计划张数 + 每张用途 + 尺寸 1080×1440**；并**写进成稿 md 末尾的 `## 生图计划` 段**（四行元信息 + ```text 提示词全文```）——这是 `xhs-notion-sync.mjs` 生成 Notion 成稿页「生图计划」块、供手机端闸门 2 审核的唯一来源。
   → **⛔ 闸门 2**：停下等用户确认或改提示词；**确认前禁止打开 ChatGPT 网页版生图**。手机端在 Notion 成稿页放行/打回，状态与 `review_note` 由同步脚本回写 Obsidian。
5. **配图**（`xiaohongshu-image-card` + `小红书配图生成-SOP`）：按用户确认后的提示词，**ChatGPT 网页版真生图**（开干净会话避免旧图、定位本次生成、缩略图要新建 Image 加载原图 → canvas→base64）→ 落盘。
6. **存附件库 + 嵌入**：PNG 存 `8️⃣ Attachment/image/`；笔记末尾「封面 / 配图」段用 `![[图名.png]]` 嵌入；emoji 路径用 Python 处理。同步脚本会把配图上传到 Notion（File Upload API），手机端才能看图审稿。
7. **呈现 + 送审**：`vision_present` 展示成稿 + 配图。
   → **⛔ 闸门 3**：置 `status: 待发布审核`，交用户决定发布或打回；打回则按 `review_note` 修订后回到相应闸门。**手机端 Notion 三道闸门的完整操作见 `2️⃣ AI/Sop/小红书Notion同步-SOP.md`。**

## 分工（复用已有 skill，不重复）

- 研究 / 成稿模板 / 素材挖掘 → `xiaohongshu-city-research`
- 配图生成链路（含 GPT web 取图坑） → `xiaohongshu-image-card` + `2️⃣ AI/Sop/小红书配图生成-SOP.md`
- 命名 / 存放 / 状态机 → `content-output-naming`
