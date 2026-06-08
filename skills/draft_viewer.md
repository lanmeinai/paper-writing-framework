# Draft Viewer Skill

## 角色
你是论文写作项目的进度可视化助手。负责将已完成的各章节草稿整合、翻译为中文，并生成带有数学渲染和学术排版的自包含 HTML 预览文件，供作者快速审阅当前写作进度和质量。

## 触发条件
- 用户说"看下写得怎么样" / "预览" / "生成 preview" / "view draft" / "整合 HTML"
- 用户直接调用 `/draft_viewer`

## 输入
- `draft/` 下所有章节文件（`01_introduction.md` ~ `06_conclusion.md`）
- `session_state.json` — 当前进度状态、术语表、引用池、字数统计
- `outline.json` — 各章节标题、目标字数、子节结构

## 输出
- `final/draft_preview.html` — 自包含 HTML 文件（MathJax CDN 外部依赖）

---

## 执行流程

### 第一步：读取所有草稿
1. 遍历 `draft/` 下所有 `.md` 文件
2. 跳过仅有占位符的章节（内容为 `# N. Title\n\n<!-- Draft content here -->` 或类似空模板）
3. 对有实质内容的章节，提取：
   - HTML 注释元数据（`section_id`、`word_count`、`status`、`citations_used`、`placeholders`）
   - 完整的 Markdown 正文

### 第二步：读取上下文
1. 读取 `session_state.json`，获取：
   - `completed_sections` — 各节摘要（summary_100_words）
   - `word_counts` — 全局字数统计
   - `key_terms_glossary` — 术语表
   - `decisions_log` — 关键决策记录
2. 读取 `outline.json`，获取各节标题映射

### 第三步：翻译与转换

将所有已完成章节的正文从英文翻译为中文，严格遵循以下规则：

#### 3.1 翻译总则
1. **意译为主**，以符合中文学术写作语境为第一原则，忠实原文技术内容，不得增删技术细节
2. **学术语体**：使用所在领域的惯用中文表达，例如"本文提出……""实验结果表明……""如图所示……"等
3. **段落格式完全保留**：原文的分段、换行、列表结构在译文中原样保持
4. **仅输出中文译文**，不保留英文原文（数学公式、术语注释、引用标记除外，见下）

#### 3.2 人名与专有名词
1. **人名一律不翻译**，保持英文原文，如 He et al.、van den Berg
2. **技术缩写与品牌名保留**，如 GPU、CNN、U-Net 等

#### 3.3 专业术语处理
1. **全文首次出现**时，写法为：中文全称 （English full term），如"生成式AI （Generative AI）"、"卷积神经网络 （Convolutional Neural Network，CNN）"
2. **首次出现之后**，只写中文全称或公认缩写，不再附英文
3. 术语认定范围：下文《术语对照表》中所有条目，以及正文中出现的其他领域专有名词
4. 纯技术缩写（已在首次出现时展开过的）后续可直接使用缩写

#### 3.4 数学公式与变量
1. 所有 `$...$`（行内公式）和 `$$...$$`（独立公式）原样保留，由 MathJax 渲染，**不翻译公式本身**
2. 公式中如含 `<`、`>`、`&`，须转义为 `&lt;`、`&gt;`、`&amp;`
3. **变量加注规则**：
   - 对具有实际物理/数学意义的变量，在**全文首次出现**时于公式后紧跟行内括号注，格式为：
     `$\mathbf{R} \in \mathbb{R}^{3\times3}$ （旋转矩阵，维度 $3\times3$）`
   - 纯计数/下标变量（$i$、$j$、$n$、$k$ 等）**不需要加注**
   - 同一变量全文只注一次，后续出现不重复加注

#### 3.5 引用标记
1. `[REF:Author_Year]` → 渲染为 `<span class="ref">[Author Year]</span>`（去掉 `REF:` 前缀，去掉下划线）
2. 数字引用 `[20]`、`[3,5]` 等原样保留，渲染为 `<span class="ref">[20]</span>`
3. **引用标记前统一加半角空格**，如"该方法 [20] 表明……"

#### 3.6 特殊标记处理
1. `[TOPIC SENTENCE: ...]` → 渲染为带"核心论点"标签的高亮段落（`.topic-sentence` 类）
2. `[PLACEHOLDER: ...]` → 渲染为红色虚线框 + 标签（`.placeholder` 类），内容翻译为中文
3. `[FIGURE: ...]` → 渲染为蓝色虚线框 + "图"标签（`.figure-placeholder` 类），说明文字翻译为中文

#### 3.7 图表标注
1. Figure → 图，Table → 表，编号保留阿拉伯数字，如 Figure 1 → 图1
2. 说明文字（caption）翻译为中文，格式为"图1：XXX"（冒号为中文全角冒号 `：`，编号与冒号之间无空格）
3. 图表内的坐标轴标签、图例文字一并翻译

#### 3.8 括号与标点规范
1. **全角括号 `（）` 统一替换为半角括号 `()`**
2. 半角左括号 `(` 前加半角空格，右括号 `)` 后加半角空格，但以下情况**除外**：
   - 句子开头的括号前不加空格
   - 括号紧接在另一个标点符号（逗号、句号、分号等）**之前**时，右括号后不加空格，由标点自然衔接
3. 术语注释括号同样适用此规则

#### 3.9 标题翻译
1. 顶级标题（`# N. Title`）→ 中文标题
2. 子标题（`## N.N`、`### N.N.N`）→ **只保留中文**，不保留英文双语

### 第四步：生成 HTML

HTML 模板结构如下，**必须严格遵循**：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>论文标题中文 — 草稿预览</title>
<script>
window.MathJax = {
  tex: { inlineMath: [['$','$'], ['\\(','\\)']], displayMath: [['$$','$$'], ['\\[','\\]']] },
  options: { ignoreHtmlClass: 'draft-comment' }
};
</script>
<script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js" async></script>
<!-- CSS 样式表直接内联 -->
<style>
  :root { /* 学术配色变量 */ }
  body { 衬线字体 最大宽度820px 居中 }
  h1,h2,h3,h4 { /* 标题样式 */ }
  .topic-sentence { 黄色左边框 高亮块 }
  .placeholder { 红色虚线框 }
  .figure-placeholder { 蓝色虚线框 }
  .ref { 蓝色内联 }
  .meta-bar { 顶部元数据条 }
  .toc { 目录区 }
  .sep { 分节线 }
  .status-draft { 黄色标签 }
</style>
</head>
<body>

<!-- 标题区 -->
<h1>论文中文标题</h1>
<p class="title-en">英文标题</p>

<!-- 元数据条 -->
<div class="meta-bar">
  <span>状态：{project_phase}</span>
  <span>已完成字数：{total}</span>
  <span>目标字数：{target}</span>
  <span>更新：{date}</span>
</div>

<!-- 目录 -->
<div class="toc">...</div>

<!-- 各章节内容按 outline 顺序依次渲染 -->

<!-- 页脚 -->
<div style="text-align:center;color:var(--muted);font-size:0.85rem;margin-top:48px;">
  <p>{论文英文标题} — Draft v1</p>
  <p>Generated {date} · Sections complete: {章节摘要} · Total: {已完成字数} / {目标字数} words</p>
</div>

</body>
</html>
```

### 第五步：写入文件
将完整 HTML 写入 `final/draft_preview.html`。

---

## CSS 设计规范

### 色彩变量
```css
:root {
  --bg: #fdfbf7;           /* 暖白背景 */
  --text: #2c2c2c;          /* 正文深灰 */
  --heading: #1a1a1a;       /* 标题黑 */
  --accent: #8b0000;        /* 暗红强调 */
  --muted: #6b6b6b;         /* 辅助灰 */
  --block-bg: #f4f1ea;      /* 区块底色 */
  --border: #d9d0c0;        /* 分隔线 */
  --topic-bg: #fff8e7;      /* 核心论点底色 */
  --topic-border: #d4a017;  /* 核心论点左边框 */
  --placeholder-bg: #fff0f0;/* 待填充底色 */
  --placeholder-border: #d44;
  --figure-bg: #f0f4ff;     /* 图表位底色 */
  --figure-border: #48b;
  --ref-color: #2a6496;     /* 引用颜色 */
  --draft-tag-bg: #e8d479;  /* 草稿标签 */
}
```

### 关键 CSS 类
| 类名 | 用途 | 特征 |
|------|------|------|
| `.topic-sentence` | 核心论点段落 | 黄色左边框 4px solid，圆角右侧，加粗字体，`::before` 伪元素标签 |
| `.placeholder` | 待填充数值 | 红色虚线边框，`::before` 标签 |
| `.figure-placeholder` | 待插入图表 | 蓝色虚线边框，居中文本，`::before` 标签 |
| `.ref` | 文献引用 | 蓝色字体，`white-space: nowrap` |
| `.meta-bar` | 顶部状态栏 | flexbox 居中，灰色背景胶囊标签 |
| `.toc` | 目录区 | 浅灰背景块，嵌套 `<ul>`，已完成项蓝色链接 |
| `.word-count` | 字数标记 | 右对齐，小号灰色 |
| `.status-draft` | 草稿状态 | 黄色背景小标签 |
| `.sep` | 章节分隔线 | 虚线上边距 |

### 布局参数
- 最大宽度：`820px`，水平居中
- 内边距：`48px 28px 80px`
- 行高：`1.85`
- 字体：优先系统宋体/衬线 → `"Noto Serif SC", "Source Han Serif SC", Georgia, serif`
- 响应式：`@media (max-width: 600px)` 缩小内边距和字号

---

## 术语对照表

翻译时须保持全文术语一致。以下为通用学术术语对照，**各项目应根据自身领域扩展此表**。所有条目在**全文首次出现**时写"中文全称 （English）"，之后只写中文全称或缩写。

### 通用学术术语

| English | 中文全称 | 后续缩写 |
|---------|----------|----------|
| Abstract | 摘要 | — |
| Introduction | 引言 | — |
| Related Work | 相关工作 | — |
| Methodology / Method | 方法论 / 方法 | — |
| Experiments | 实验 | — |
| Discussion | 讨论 | — |
| Conclusion | 结论 | — |
| Ablation Study | 消融实验 | — |
| Baseline | 基线方法 | — |
| Benchmark | 基准测试 | — |
| State-of-the-art (SOTA) | 当前最优 | SOTA |
| Ground Truth | 真值 | — |
| Loss Function | 损失函数 | — |
| Feature Extraction | 特征提取 | — |
| Encoder / Decoder | 编码器 / 解码器 | — |
| Attention Mechanism | 注意力机制 | — |
| Transformer | Transformer | — |
| Convolutional Neural Network (CNN) | 卷积神经网络 | CNN |
| Generative Adversarial Network (GAN) | 生成对抗网络 | GAN |
| Recurrent Neural Network (RNN) | 循环神经网络 | RNN |
| Multilayer Perceptron (MLP) | 多层感知机 | MLP |
| Stochastic Gradient Descent (SGD) | 随机梯度下降 | SGD |
| Cross-Validation | 交叉验证 | — |
| Overfitting / Underfitting | 过拟合 / 欠拟合 | — |
| Hyperparameter | 超参数 | — |
| Regularization | 正则化 | — |
| Normalization | 归一化 / 标准化 | — |
| Data Augmentation | 数据增强 | — |
| Transfer Learning | 迁移学习 | — |
| Fine-tuning | 微调 | — |
| End-to-end | 端到端 | — |
| Coarse-to-fine | 由粗到精 | — |
| Real-time | 实时 | — |
| Inference | 推理 | — |
| Training / Validation / Test Set | 训练集 / 验证集 / 测试集 | — |
| Precision / Recall / F1 Score | 精确率 / 召回率 / F1值 | — |
| Mean Absolute Error (MAE) | 平均绝对误差 | MAE |
| Root Mean Square Error (RMSE) | 均方根误差 | RMSE |
| Computational Complexity | 计算复杂度 | — |
| Time Complexity | 时间复杂度 | — |
| Memory Footprint | 内存占用 | — |
| Throughput | 吞吐量 | — |
| Latency | 延迟 | — |
| Robustness | 鲁棒性 | — |
| Generalization | 泛化能力 | — |
| Convergence | 收敛 | — |
| Gradient Descent | 梯度下降 | — |
| Backpropagation | 反向传播 | — |

> **领域扩充说明**：各项目启动时应根据自身领域（如计算机视觉、自然语言处理、生物信息学、材料科学等）将领域专属术语追加至本表。`draft_viewer` 生成 HTML 时会自动参考本表进行术语翻译。在翻译过程中遇到上表未收录的专业术语，按 3.3 节规则处理，并在完成后将新术语追加至本表，以保持全文一致性。

---

## HTML 生成约束

1. **数学公式**：所有 `$...$` / `$$...$$` 原样保留，由 MathJax CDN 渲染。确保公式中无未转义的 HTML 特殊字符（`<`、`>`、`&` 在公式中需转义为 `&lt;`、`&gt;`、`&amp;`）
2. **目录生成**：必须列出 `outline.json` 中所有章节，已完成的用蓝色可点击链接（`href="#sN"`），未完成的用灰色文字
3. **章节锚点**：每个 `<h2>` 和 `<h3>` 需设置 `id` 属性（如 `id="s2"`、`id="s3.1"`），与目录链接对应
4. **占位符处理**：所有 `[PLACEHOLDER: ...]` 和 `[FIGURE: ...]` 必须可见，方便作者识别哪些内容待补充
5. **引用渲染**：
   - `[REF:Author_Year]` → `<span class="ref">[Author Year]</span>`
   - `[20]`、`[3,5]` 等数字引用 → `<span class="ref">[20]</span>`
   - 引用标记前统一有半角空格
6. **章节编号**：h2 编号保持 `2.`、`3.` 等，h3 编号保持 `2.1`、`3.1` 等，与 outline.json 一致
7. **文件命名**：始终输出到 `final/draft_preview.html`（覆盖旧版）
8. **编码**：UTF-8，`<meta charset="UTF-8">`
9. **更新 `session_state.json`**：在 `decisions_log` 中记录本次 HTML 生成

---

## 翻译自检清单

生成 HTML 前，对译文逐项核查：

- [ ] 全角括号已全部替换为半角括号，空格规则符合 3.8 节
- [ ] 术语对照表中所有条目首次出现均已加注英文，后续不重复
- [ ] 数学变量（有实际含义者）首次出现已在公式后行内加注
- [ ] 人名未被翻译
- [ ] 引用标记前有半角空格
- [ ] 图表标注格式为"图N：……"
- [ ] 无全角括号残留
- [ ] 新出现的专业术语已追加至术语对照表

---

## 使用示例

> **User**: /draft_viewer
>
> **Claude**:
> 扫描 `draft/` 目录... 发现 2 个已完成章节（§2 相关工作 1200词、§3 方法论 3500词），4 个待撰写。
> 翻译中...
> HTML 生成完毕 → `final/draft_preview.html`
> 总进度：4,700 / 10,000 词（47.0%）
