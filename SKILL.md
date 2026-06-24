---
name: exam-flashcard-builder
description: "Build an interactive HTML flashcard review system for ANY subject from uploaded study materials. Reads source files (PDF / DOCX / PPTX / Markdown / TXT / images), auto-analyzes key points and difficulty, distills them into difficulty-graded flip-cards (基础/进阶/重难点), and generates extended practice — auto multiple-choice quizzes and confusable-point comparison (易混辨析). The generated app uses a cute Animal-Crossing-style UI (left sidebar nav, fixed no-scroll viewport, right-corner floating tools, dark mode) and renders math formulas (KaTeX) and syntax-highlighted code (highlight.js). Triggers include: '复习', '备考', '笔试复习', '考研复习', '期末复习', '蒸馏复习资料', '制作复习卡片', '把这份资料做成卡片', 'flashcard', 'exam prep', 'study cards', '帮我复习这门课', '上传资料做复习系统'. Works for political theory, law, medicine, finance/economics, CS (code), math/physics (formulas), languages, certifications — any memorization-heavy subject."
---

# Exam Flashcard Builder（通用复习卡片系统生成器）

## Overview

从**任意科目**的复习资料生成一个自包含、可离线运行的 HTML 复习系统。沿用固定的精致界面（Animal Crossing 风），换科目只需替换数据。

**界面与交互（模板已内置，勿改结构）：**
- **左侧边栏导航**：品牌 + 模式切换（📖学习 / 📝测验 / ⚖️辨析）+ 深色切换；顶部不拥挤
- **自适应视口、整页不滚动**（`height:100vh; overflow:hidden`，内容区 flex 填充，超长内容内部滚动）
- **右下角浮动按钮（FAB）**：↺重置 / ✍️默写 / 🖌️悬浮手写
- **学习模式**：3D 翻转卡，按难度分级（基础/进阶/重难点），进度持久化
- **默写练习**：点 ✍️ 在页面内**左右分屏**——左为可翻转卡片（看题目↔答案），右为默写区（打字/手写）；左右**等高**；**切换卡片自动清空**默写
- **测验模式**：卡片自动出单选题，即时判分、错题自动进**错题本**，可"全部/仅重难点/仅错题"
- **辨析模式（易混辨析）**：自动识别或人工标注的**易混知识点并排对比**，也可搜索任意卡片自由组队
- **数学公式 + 代码**：卡片内容支持 `$行内$` / `$$块级$$` 公式（KaTeX）与 ```` ```语言 ```` 代码块（highlight.js 高亮）、`` `行内代码` ``
- 重难点筛选、深色模式、搜索、手写画板（已修复笔迹坐标 bug）、键盘快捷键

> 配色对齐 Animal Island（暖近白底 `#faf9f3` + 纯白卡 + 棕字 `#725d42` + 薄荷绿 `#19c8b9`），柔和不刺眼。

## When to Use

- 用户上传/指向复习资料，想做成卡片复习系统
- 用户说"帮我复习 X""把这份资料做成复习卡片""蒸馏重难点""做套题练习"
- 准备任何闭卷/开卷考试、考研、期末、资格考试、面试笔试
- 内容含**公式（数学/物理/金融）或代码（计算机）**，需要正确排版

## Workflow

### Step 1 — 明确科目与目标
1. 科目/考试名称、范围、题型、闭卷/开卷、时间
2. 是否有**难度标准来源**（考纲、真题、教材重点标注）
3. 系统标题/副标题（替换 `{{SYSTEM_TITLE}}` / `{{SYSTEM_SUBTITLE}}`）

### Step 2 — 读取并解析资料
| 类型 | 方式 |
|------|------|
| `.docx` | python-docx 读 `paragraphs` 与 `tables` |
| `.pdf` | pdfplumber / PyMuPDF；扫描件 OCR |
| `.pptx` | python-pptx 遍历 slides/shapes |
| `.md`/`.txt` | 直接读取 |
| 图片 | 视觉理解 / OCR |
| 目录批量 | Glob `**/*.{pdf,docx,pptx,md,txt}` |

时效性内容（时政/法规/指南）用 WebSearch 补最新版本。

### Step 3 — 重难点智能分析与难度分级（核心）
为每个知识点判定 `level`（1基础 / 2进阶 / 3重难点）。**优先级灵活组合：**
1. **资料自带标准（最高）**：考纲权重、教材★/重点/必背、真题频次 → 直接定级
2. **真题反推**：高频/易错考点 → 2–3
3. **AI 综合判断**：抽象度、记忆量、易混度、体系地位
4. **联网校准**：把握不准时 WebSearch "<科目> 高频考点/难点/易错点"

> 模板也内置 `autoLevel` 启发式：未标 `level` 的卡片按内容长度/数字型列表自动定级。建议仍尽量显式标注。

### Step 4 — 整理卡片数据
按模块分牌组（deckKey 用 ASCII），一卡一知识点。数据格式见 `references/card-data-spec.md`：
```javascript
deckKey: {
    name: '牌组名',
    isNew: true,                 // 可选高亮
    cards: [
        { title: '知识点', content: '表述\n用 \\n 换行', level: 3 }
    ]
}
```
**公式/代码**写在 `content` 里：`$E=mc^2$`、`$$\int_a^b f$$`、以及
```` ```python\nprint("hi")\n``` ````（注意 JS 字符串里换行用 `\n`）。

### Step 5 — 生成扩展练习
- **选择题**：默认 `quizBank = []` → 卡片自动出题；重点科目可精编 `quizBank`（题干+强干扰项+解析）
- **易混辨析**：默认 `confusables = []` → 按标题前缀自动识别；可精编 `confusables` 精准成组（`items` 填已存在卡片 title）

### Step 6 — 生成 HTML
1. 读 `assets/flashcard-template.html`
2. 替换占位符：
   - `{{SYSTEM_TITLE}}`（`<title>` 与侧边栏 `.brand-title` 两处）
   - `{{SYSTEM_SUBTITLE}}`（`.brand-sub`）
   - `<<<DECK_DATA_START>>> … <<<DECK_DATA_END>>>` 内的 `var decks = {…}`
   - `var quizBank = []; // <<<QUIZ_BANK>>>`（可选）
   - `var confusables = []; // <<<CONFUSABLES>>>`（可选）
3. 写入 `<科目>_复习卡片系统.html`

### Step 7 — 验证
浏览器/present_files 打开，确认：侧边栏模式切换、翻转、难度星级、重难点/错题筛选、测验判分+错题本、辨析并排、分屏默写（左右等高、切卡自动清空）、深色、整页不滚动、**公式与代码正确渲染**。可用 Playwright headless 跑冒烟测试（见 `references/distillation-workflow.md`）。

## 难度分级标准
| level | 标签 | 含义 |
|-------|------|------|
| 1 | 基础 | 易记、低频 |
| 2 | 进阶 | 需理解、中频 |
| 3 | 重难点 | 高频 + 易错 + 难记（自动进"重难点"筛选）|

## 技术约束（必须遵守）
1. **所有 JS 标识符用 ASCII**——deckKey、变量名、DOM id 不能用中文（中文只在字符串值里）
2. 换行用字符串内的 `\n`
3. 替换 `decks`/`quizBank`/`confusables` 时保持 JS 语法合法（逗号/括号/引号配平）
4. **不要改动模板的界面结构、CSS 类名、`<script>` 逻辑**——只替换占位符与数据，保证"换科目仍是原样界面"
5. 公式/代码依赖 KaTeX、highlight.js CDN；离线时公式显示为原文 `$...$`、代码为纯等宽（仍可读），不影响其它功能

## Resources
- `assets/flashcard-template.html` —— 完整模板引擎（侧边栏 + 三模式 + 难度分级 + 分屏默写 + 手写 + 深色 + 公式/代码渲染）。仅替换占位符与数据。
- `references/card-data-spec.md` —— `decks` / `quizBank` / `confusables` 数据结构与公式/代码写法
- `references/distillation-workflow.md` —— 解析、分级、练习生成、验证完整流程
