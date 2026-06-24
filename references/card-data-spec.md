# 数据结构规范（通用复习卡片系统）

模板内有三个可替换数据结构：`decks`（必填）、`quizBank`（可选）、`confusables`（可选）。
替换时保持 JavaScript 语法合法，**所有标识符（deckKey、变量名）用 ASCII**，中文只出现在字符串值里。

## 一、牌组数据 `decks`（必填）

```javascript
var decks = {
    deckKey: {                       // ASCII，如 core / keypoints / algo1
        name: '牌组显示名称',         // 中文可
        isNew: true,                 // 可选：高亮为"新增"牌组
        cards: [
            {
                title: '卡片标题（正面，概念名/问题）',
                content: '卡片内容（背面）\n\n用 \\n 换行\n支持多段落',
                level: 3              // 可选：1=基础 2=进阶 3=重难点
            }
        ]
    }
};
```

| 字段 | 必填 | 说明 |
|------|------|------|
| `name` | 是 | 牌组按钮名 |
| `cards[].title` | 是 | 卡片正面，简明 |
| `cards[].content` | 是 | 卡片背面，完整可背诵，`\n` 换行 |
| `cards[].level` | 否 | 难度 1/2/3；标 3 自动进"重难点"并显示红色星级；不标则由 `autoLevel` 估算 |
| `isNew` | 否 | 牌组高亮 |

## 二、公式与代码（写在 title / content 字符串里）

模板用 KaTeX 渲染数学、highlight.js 渲染代码。

### 数学公式
- 行内：`$...$`，例如 `'欧姆定律 $V = IR$ 描述……'`
- 块级：`$$...$$`，例如 `'$$\\sum_{i=1}^{n} i = \\frac{n(n+1)}{2}$$'`
- **注意 JS 字符串里反斜杠要写两个**：`\\frac`、`\\log`、`\\pi`、`\\sum`

### 代码
- 围栏代码块（带语言名，自动高亮）：
  ```javascript
  { title: '快速排序', content: '分治思想：\n\n```python\ndef qsort(a):\n    if len(a) <= 1: return a\n    p = a[0]\n    return qsort([x for x in a[1:] if x < p]) + [p] + qsort([x for x in a[1:] if x >= p])\n```' }
  ```
  即 content 里出现 ```` ```python ````、换行 `\n`、代码、再 ```` ``` ````。
- 行内代码：`` `like_this` ``，例如 `'调用 `np.mean()` 求均值'`

> 离线（无网络）时：公式显示为原文 `$...$`，代码显示为纯等宽文本，仍可读，不影响其它功能。

## 三、精编选择题 `quizBank`（可选）

留空 `[]` 时系统从卡片**自动生成**单选题。需要高质量题目时填充：

```javascript
var quizBank = [
    {
        q: '题干（可含 $公式$ 或代码）',
        options: ['选项A', '选项B', '选项C', '选项D'],
        answer: 1,                   // 正确项下标（0 起）
        explain: '解析：……',        // 可选，答错时展示（支持公式/代码）
        key: 'q1'                    // 可选，错题本去重
    }
];
```
`quizBank` 非空时，测验模式全部用该题库（全局）。

## 四、易混辨析组 `confusables`（可选）

留空 `[]` 时按标题前缀（"四个…""三大…"）自动识别。精准控制时填充：

```javascript
var confusables = [
    { title: '现值 vs 终值', items: ['现值的定义', '终值的定义'] }
];
```
`items` 填**已存在卡片的 title**（可跨牌组），系统自动并排展示其内容（含公式/代码）。每组 2–4 项。

## 五、关键约束
1. 标识符（deckKey、变量名、DOM id）一律 **ASCII**
2. 换行用字符串内的 `\n`；公式反斜杠写 `\\`
3. 替换后检查逗号/括号/引号配平，确保 `<script>` 是合法 JS
4. localStorage key 带 `_v2` 后缀，结构升级改后缀即可重置进度
5. 占位符替换之外**不改模板结构/CSS/JS**，确保换科目仍是同一界面
