# 小红书内容生产工作流

> 一套把「城市选择 / 宜居决策」类小红书内容，从**痛点选题 → 研究与成稿 → 数据配图 → 命名归档**跑完的、可复用的 Agent 技能集。核心关键词：**数据可核验、选题痛点驱动、表达克制**。

这套工作流不是灵感摆件，而是从长期写「城市选择」类小红书内容（60+ 篇）里一步步沉淀、反复校验出来的实操方法。用 Agent（DeepSeek Harness / Claude Code / Codex 等）运行，把「怎么研究」「怎么保证数字不编」「怎么写才有人看」「图怎么配」拆成可直接照做的指令。

## 特性 / 硬规则

- **生图只用 ChatGPT 网页版真生图**。禁用 Codex 生图、禁用 HTML/CSS 渲染生图、禁用其它任何生图工具。配图提示词先经 `gpt-image-2-style-library`（外部库）产出，再走 chatgpt.com 的真·图片生成。
- **选题必须「痛点驱动、具体可感」**。打特定人群的具体痛点（收入焦虑 / 房租成本 / 存钱 / 通勤 / 选择困难 / 婚恋生育 / 落户门槛），能让人一眼对号入座。**禁止**宽泛抽象的「元问题 / 框架类」选题（如「XX 值不值得去」「城市怎么选」这类没人点）。
- **数据诚实**。来源可核验；关键数字区分「官方原始数据 / 权威转述 / 派生值 / 估算值 / 待核实」并标证据等级；拿不准标「待核实」，不编造精确数字、不用「2026 最新 / 精确排名」包装缺依据的模型结果。
- **内容命名与归档统一**。日期开头 + 实词关键词的命名规则、系列文件夹的拆分阈值、`status` 状态机（`草稿 → 打磨中 → 待发布 → 已发布`）原地在原地推进，不搬文件。
- **配图审美以「数据呈现图」为主**：深绿 `#2E7D5B` 品牌 + 红橙紫蓝青绿阶梯多彩数据条形/饼图/地图/排名卡，竖版 3:4，多模块（大标题 + 数据源条 + 主图 + 数据分析 + 一句话总结 + 页脚署名）；图内**不放 `#` 话题标签**。

## 仓库结构

```
xiaohongshu-content-workflow/
├── README.md                       # 本文件
├── LICENSE                         # MIT
├── skills/                         # Agent 技能指令（SKILL.md，可直接加载）
│   ├── xiaohongshu-content-pipeline/   # 内容生产一条龙：痛点选题→研究→成稿→配图→落地
│   ├── xiaohongshu-city-research/      # 城市研究方法 + 数据可核验 + 选题雷达 + 成稿模板
│   ├── xiaohongshu-image-card/         # 图卡 / 配图生成：style-library→ChatGPT 网页版→存附件库→嵌入
│   └── content-output-naming/          # 自媒体内容命名 / 系列归档 / 状态机
├── docs/
│   └── 小红书配图生成-SOP.md         # 配图生成的逐步 SOP 与踩坑（含 AppleScript + Chrome 取图细节）
└── examples/
    └── 2026-09-06-一线房租连涨-月薪8000撑得住吗.md   # 一则真实成稿示例（痛点驱动 + 数据口径标注）
```

## Skills 说明

| Skill | 作用 | 关键点 |
| --- | --- | --- |
| `xiaohongshu-content-pipeline` | 端到端流水线 | 确认痛点 → 研究 → 成稿 → 配图 → 存附件 → 呈现 |
| `xiaohongshu-city-research` | 研究底座 | 固定口径、来源优先级、证据标签、生活成本模型、可比性检查、选题评分、成稿/图卡模板、选题雷达 |
| `xiaohongshu-image-card` | 配图生成 | style-library 出提示词 → ChatGPT 网页版真生图 → 存附件库 → 笔记嵌入 |
| `content-output-naming` | 命名归档 | `YYYY-MM-DD-关键词1-关键词2`、系列文件夹、`status` 状态机 |

## 如何使用

把它当作一套 **Agent 技能（Skill）** 使用：

1. 把 `skills/<skill名>/SKILL.md` 放到你 Agent 的技能目录（如 `~/.agents/skills/<skill名>/SKILL.md`，或自定义 `customSkillDirs`）。
2. 触发对应任务时（如「研究一下 XX 城市」「给这篇配个图」）Agent 会自动加载并遵循。
3. 分支依赖：

| 依赖 | 说明 |
| --- | --- |
| `gpt-image-2-style-library` | 配图提示词模板库（**外部依赖**，源自 [`freestylefly/awesome-gpt-image-2`](https://github.com/freestylefly/awesome-gpt-image-2) 风格库的封装；`infographic-engine` / `scene-storytelling` 等模板由它提供） |
| Obsidian（可选） | `content-output-naming` 与成稿落盘假设用一个 Obsidian vault（见下「适配」），非 Obsidian 也可只用其命名/状态规则 |
| Chrome + ChatGPT 网页版 | `xiaohongshu-image-card` 走 chatgpt.com 真生图，需 Chrome 开启「查看 → 开发者 → 允许 Apple 事件中的 JavaScript」 |

## 如何适配到你的账号 / 目录

这套工作流里有两类「账号级」配置，**都是可替换的示例**，按你自身情况改即可：

- **账号人格（默认示例）**：`Light｜普通人城市选择指南`（🧠 拆解城市资源 / 工作机会 / 薪资水平；🏙️ 商场·配套·便利度真实记录；💰 帮你判断「这座城市值不值得去」）。分布在 `xiaohongshu-city-research`（素材挖掘 / 成稿署名）与成稿示例里。换成你自己的账号定位即可。
- **Obsidian 目录假设**：命名 / 归档 / 配图落盘假设一个带 emoji 编号的 Obsidian vault（`3️⃣ output/media/<平台>/`、`8️⃣ Attachment/image/`、`2️⃣ AI/Sop/`、`1️⃣ input/`）。非 Obsidian 用户可按同样规则映射到自己的媒体 / 附件 / 笔记目录。
- **设计规范（默认示例）**：数据卡用深绿 `#2E7D5B` 品牌 + 阶梯多彩数据色；纯情感非数据生活向才用暖色生活场景。属个人审美偏好，可整体替换。

## License

[MIT](LICENSE)
