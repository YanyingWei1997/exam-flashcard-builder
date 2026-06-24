# 📚 exam-flashcard-builder

**EN** · A Claude Code / Codex **Skill** that turns study materials for **any subject** into a self-contained, offline-capable interactive flashcard review system — auto difficulty grading, quizzes, confusable-point comparison, split-screen dictation, math formulas and code highlighting.

**中文** · 一个 Claude Code / Codex **技能（Skill）**：上传任意科目的资料（PDF / DOCX / PPTX / Markdown / TXT / 图片），自动蒸馏重难点、分级，并生成一个**单文件 HTML 复习系统**。换科目只需替换数据，界面始终如一。

<p align="center">
  <img src="screenshots/01-formula.png" width="80%" alt="flashcard review system">
</p>

---

## ✨ Features / 功能特性

| Feature 功能 | Description / 说明 |
|------|------|
| 📖 Study / 学习 | 3D flip cards graded by difficulty · 3D 翻转卡片，按难度分级（基础/进阶/重难点）|
| 📝 Quiz / 测验 | Auto-generated multiple choice with instant scoring + wrong-answer book · 自动单选题，即时判分，答错进**错题本** |
| ⚖️ Compare / 辨析 | Side-by-side comparison of confusable points · 易混知识点**并排对比**（自动识别 + 可手动指定）|
| ✍️ Dictation / 默写 | In-page split screen: flippable card on the left, type/handwrite on the right · **左右分屏**，左可翻转卡片、右默写区，切卡自动清空 |
| 🧮 Formula + Code / 公式+代码 | `$LaTeX$` math via KaTeX + syntax-highlighted code via highlight.js · 数学公式 + 代码语法高亮 |
| ⭐ Difficulty / 难度分级 | Auto / manual grading with filtering · 自动/手动标注，支持按难度筛选 |
| 🌙 Dark mode / 深色模式 | Warm night theme · 暖色夜间主题 |
| 📐 Adaptive / 自适应 | Left sidebar nav, no-scroll viewport, floating tools, mobile-friendly · 侧边栏导航、整页不滚动、浮动工具、移动端友好 |
| 💾 Persistence / 进度保存 | Progress / wrong answers / theme saved in localStorage · 本地保存掌握/错题/主题 |

| Code highlight 代码高亮 | Quiz 测验 | Dark mode 深色模式 |
|:---:|:---:|:---:|
| ![](screenshots/02-code.png) | ![](screenshots/03-quiz.png) | ![](screenshots/04-dark.png) |

| Dictation · typing 分屏默写·打字 | Dictation · handwriting 分屏默写·手写 |
|:---:|:---:|
| ![](screenshots/05-dictation-typing.png) | ![](screenshots/06-dictation-handwriting.png) |

> UI inspired by [Animal Island UI](https://github.com/guokaigdg/animal-island-ui) (Animal Crossing style) · 界面风格参考 Animal Island UI（Animal Crossing 风）。

---

## 🚀 Install / 安装

**EN** · Compatible with both **Claude Code** and **OpenAI Codex CLI** (both use the `SKILL.md` convention).
**中文** · 同时兼容 **Claude Code** 与 **OpenAI Codex CLI**（二者均采用 `SKILL.md` 规范）。

**Claude Code** — clone into the user skills dir (auto-discovered) / 克隆到用户 skills 目录（自动发现）：

```bash
git clone https://github.com/YanyingWei1997/exam-flashcard-builder.git \
  ~/.claude/skills/exam-flashcard-builder
```

**OpenAI Codex CLI** — Skills supported since 2025-12, clone into `~/.codex/skills` / Skills 自 2025-12 起支持：

```bash
git clone https://github.com/YanyingWei1997/exam-flashcard-builder.git \
  ~/.codex/skills/exam-flashcard-builder
```

> Codex can also install from a repo URL via the built-in `$skill-installer`; for project-level sharing put it in the repo's `.codex/skills/`, and toggle it in `~/.codex/config.toml` with `[[skills.config]]`.
> Codex 也可用内置 `$skill-installer` 按仓库 URL 安装；项目级共享放入仓库 `.codex/skills/`，启停在 `~/.codex/config.toml` 的 `[[skills.config]]` 管理。

**EN** · After installing, just say *"turn this material into flashcards"* / *"build a review system from this course"*.
**中文** · 安装后在对话中说「**帮我把这份资料做成复习卡片**」「**用这门课的资料做个复习系统**」即可触发。

---

## 🧠 Workflow / 工作流

| Step | EN | 中文 |
|------|----|----|
| 1 | Parse PDF / DOCX / PPTX / MD / TXT / images | 解析资料 |
| 2 | Grade key points by syllabus / past exams / AI / web search | 重难点分级（基础/进阶/重难点）|
| 3 | Distill into decks, one point per card (formulas & code supported) | 整理卡片，一卡一知识点 |
| 4 | Auto quiz + confusable comparison (or curated `quizBank` / `confusables`) | 扩展练习：选择题 + 易混辨析 |
| 5 | Fill template placeholders → output `<subject>_review.html` | 生成单文件 HTML 复习系统 |

## 📁 Structure / 结构

```
exam-flashcard-builder/
├── SKILL.md                          # Skill definition & workflow / 技能定义与工作流
├── assets/
│   └── flashcard-template.html       # Full template engine / 完整模板引擎
├── references/
│   ├── card-data-spec.md             # Data format (cards / quiz / compare, formula & code) / 数据格式
│   └── distillation-workflow.md      # Parse → grade → generate → verify / 完整流程
└── screenshots/
```

## 🛠️ Tech / 技术

Pure vanilla HTML / CSS / JS, single file, zero build · 纯原生，单文件零依赖。
Math: [KaTeX](https://katex.org/) · Code: [highlight.js](https://highlightjs.org/) (CDN; gracefully degrades to plain text offline / 离线降级为纯文本).

## 📄 License

MIT
