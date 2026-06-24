# 内容蒸馏工作流（通用版）

从上传资料 → 重难点分级 → 卡片 + 扩展练习 → 生成 HTML。适用任意科目（含公式/代码）。

## 第一步：明确科目与目标
1. 科目/考试名称、范围、题型、闭卷/开卷、时间
2. 是否有难度标准来源（考纲、真题、教材重点标注）
3. 系统标题与副标题（替换 `{{SYSTEM_TITLE}}` / `{{SYSTEM_SUBTITLE}}`）

## 第二步：解析上传资料

### docx
```python
from docx import Document
doc = Document('file.docx')
for p in doc.paragraphs:
    if p.text.strip(): print(p.text)
for tbl in doc.tables:
    for row in tbl.rows:
        print(' | '.join(c.text for c in row.cells))
```
### pdf
```python
import pdfplumber
with pdfplumber.open('file.pdf') as pdf:
    for page in pdf.pages:
        print(page.extract_text() or '')
```
扫描件 → OCR / 视觉理解。

### pptx
```python
from pptx import Presentation
for slide in Presentation('file.pptx').slides:
    for shape in slide.shapes:
        if shape.has_text_frame: print(shape.text_frame.text)
```
### 批量
Glob `**/*.{pdf,docx,pptx,md,txt}`；重点：真题、考纲、重点汇编。时政/法规/指南用 WebSearch 补最新。

## 第三步：重难点分级（按优先级灵活组合）
1. 资料自带标准（考纲权重、★/重点、真题频次）→ 直接定级
2. 真题反推：高频/易错 → 2–3
3. AI 综合判断：抽象度、记忆量、易混度、体系地位
4. 联网校准：WebSearch "<科目> 高频考点/难点/易错点"

映射：1基础 / 2进阶 / 3重难点。未标 `level` 时模板 `autoLevel` 兜底。

## 第四步：整理卡片数据
按模块分牌组（deckKey ASCII），每知识点一卡，标 `level`。
- **公式**写 `$..$` / `$$..$$`（JS 串里反斜杠写 `\\`）
- **代码**写 ```` ```语言\n...\n``` ````、行内 `` `code` ``
- **易混点**在 content 并列差异，或用 `confusables`

格式见 `card-data-spec.md`。

## 第五步：生成扩展练习
- 选择题：默认 `quizBank=[]` 自动出题；重点科目精编 `quizBank`
- 易混辨析：默认 `confusables=[]` 自动识别；可精编 `confusables`（items 填已存在卡片 title）

## 第六步：生成 HTML
1. 读 `assets/flashcard-template.html`
2. 替换：
   - `{{SYSTEM_TITLE}}`（`<title>` 与侧边栏 `.brand-title` 两处）
   - `{{SYSTEM_SUBTITLE}}`（`.brand-sub`）
   - `<<<DECK_DATA_START>>> … <<<DECK_DATA_END>>>` 内 `var decks = {…}`
   - `var quizBank = []; // <<<QUIZ_BANK>>>`
   - `var confusables = []; // <<<CONFUSABLES>>>`
3. 输出 `<科目>_复习卡片系统.html`

> 只替换占位符与数据，**不改界面结构/CSS/JS**，确保换科目仍是同一精致界面。

## 第七步：验证
浏览器打开，逐项确认：侧边栏模式切换、翻转、难度星级、重难点/错题筛选、测验判分+错题本、辨析并排、**分屏默写（左右等高、切卡自动清空）**、深色、整页不滚动、**公式与代码渲染**。

### Playwright headless 冒烟测试（可选）
```javascript
const { chromium } = require('playwright');
(async () => {
  const b = await chromium.launch(); const p = await b.newPage();
  const errs=[]; p.on('pageerror',e=>errs.push(e.message));
  await p.goto('file://'+require('path').resolve('out.html'));
  await p.waitForTimeout(1800);                                  // 等 KaTeX/hljs CDN
  console.log('decks', await p.locator('.deck-btn').count());
  console.log('noScroll', !(await p.evaluate(()=>document.documentElement.scrollHeight>window.innerHeight+2)));
  await p.locator('#btnFlip').click(); await p.waitForTimeout(500);
  console.log('katex', await p.locator('#backContent .katex').count());   // 含公式时 >0
  await p.locator('.mode-btn[data-mode-view="quiz"]').click();
  await p.locator('.quiz-opt').first().click();                 // 判分
  console.log('errors', errs);                                   // 应为 []
  await b.close();
})();
```
先 `node --check` 抽出的 `<script>` 确保 JS 语法合法，再跑交互测试。

## 通用考点清单（按科目挑选）
- 理论/政策：核心思想、最新会议/文件、重要论述、数字型表述
- 法律：核心法条、构成要件、易混罪名、最新修订
- 医学：诊断标准、鉴别诊断（天然易混）、最新指南、剂量数值
- 财经/金融：定义与**公式**、模型假设、计量方法、易混概念
- 计算机：算法复杂度（**公式**）、**代码**实现、协议/结构对比、易混 API
- 数学/物理：定理与**公式推导**、适用条件、易错点
- 语言：词汇、语法点、易混词辨析、固定搭配
