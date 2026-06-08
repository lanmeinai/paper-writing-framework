# Paper Writing Framework

> 一个**领域无关**的学术论文写作框架 — 将 AI 辅助论文撰写流程标准化，适配计算机科学、工程、自然科学、医学、社会科学等任何领域。

---

## 这是什么？

Paper Writing Framework 是一套基于 Claude Code 的**结构化论文写作系统**。它将论文撰写过程拆分为 5 个协作 Skill（上下文管理、章节写作、文献阅读、引用管理、草稿预览），通过 3 个 JSON 状态文件（大纲、知识库、会话状态）实现跨会话的持久化进度追踪。

本框架源自 [paper-project](https://github.com/lanmeinai/paper-project)（一篇肝脏点云配准论文的完整写作项目），经**完全泛化**后适配任何学科领域。

### 核心理念

```
输入材料（PDF / 专利 / 笔记）
    │
    ▼
paper_reader ──→ knowledge_base.json    ← 结构化文献数据库
    │
    ▼
outline.json ──→ paper_writer           ← 按大纲逐章撰写
    │
    ▼
ref_manager ──→ references.bib          ← 自动检索 + 格式化引用
    │
    ▼
draft_viewer ──→ draft_preview.html     ← 中文学术预览
    │
    ▼
context_manager ──→ session_state.json  ← 跨会话进度持久化
```

---

## 文件架构

```
paper-writing-framework/
│
├── README.md                          # 本文件
├── PROGRESS_REPORT.md                 # 人类可读的进度报告（自动生成）
│
├── outline_template.json              # 论文大纲模板 → 使用前复制为 outline.json
├── knowledge_base_template.json       # 知识库模板 → 使用前复制为 knowledge_base.json
├── session_state_template.json        # 会话状态模板 → 使用前复制为 session_state.json
│
├── skills/                            # 写作助手 Skill 定义（5 个）
│   ├── context_manager.md             # 状态管理 Skill（checkpoint / 恢复 / 进度报告）
│   ├── paper_writer.md                # 章节写作 Skill（SCI 学术规范）
│   ├── paper_reader.md                # 文献阅读 Skill（PDF → 结构化知识库）
│   ├── ref_manager.md                 # 参考文献管理 Skill（Semantic Scholar API + BibTeX）
│   └── draft_viewer.md                # HTML 预览生成 Skill（中文翻译 + MathJax 渲染）
│
├── draft/                             # 论文草稿（Markdown，按章节拆分）
│   ├── 01_introduction.md
│   ├── 02_related_work.md
│   ├── 03_methodology.md
│   ├── 04_experiments.md
│   ├── 05_discussion.md
│   └── 06_conclusion.md
│
├── final/                             # 最终产出
│   ├── paper_v1.md                    # 合并全文
│   ├── abstract.md                    # 结构化摘要
│   ├── draft_preview.html             # 中文学术预览（MathJax 渲染）
│   └── submission_checklist.md        # 投稿检查清单
│
├── figures/                           # 论文插图
│   ├── *.png / *.svg                  # 图表文件
│   └── pipeline_overview.txt          # 方法流程图指南
│
├── experiments/                       # 实验代码（按项目自定义）
│   ├── run_all.py                     # 主实验套件
│   ├── requirements.txt               # Python 依赖
│   └── results/                       # 实验结果
│
├── references/                        # 参考文献
│   ├── references.bib                 # BibTeX 条目
│   └── references_candidates.json     # 候选文献检索结果
│
├── inputs/                            # 原始输入材料
│   ├── references/                    # 论文 PDF
│   └── patents/                       # 专利文件
│
└── tools/                             # 辅助工具脚本
    └── *.py                           # 按需添加
```

---

## 5 个 Skill 详解

### 1. context_manager — 上下文管家

**职责**：跨会话状态持久化，防止上下文溢出导致工作丢失。

**触发时机**：
- 每次新会话启动（最高优先级，必须先执行）
- 上下文占用超过 60%（自动 checkpoint）
- 每个章节完成后（更新进度）

**输入/输出**：
| 输入 | 输出 |
|------|------|
| `session_state.json`、`outline.json`、`draft/*.md` | 更新后的 `session_state.json` + `PROGRESS_REPORT.md` |

**核心能力**：
- 新会话启动时自动恢复进度并输出摘要
- 维护术语表保证全文术语一致
- 维护引用池防止重复引用
- 记录所有关键决策及理由
- 自动生成人类可读的进度报告

---

### 2. paper_writer — 章节写手

**职责**：按大纲撰写指定章节的英文初稿，遵守 SCI 学术规范。

**核心约束**：
- **禁止使用 "we propose"**，统一被动语态
- 每段以 `[TOPIC SENTENCE]` 开头
- 引用格式 `[REF:Author_Year]`
- **绝对禁止捏造实验数据** — 无数据时标记 `[PLACEHOLDER]`
- 术语全篇统一，首次出现标记 `\textit{}`

**执行流程**：
1. 加载 `outline.json` → 定位目标章节
2. 筛选 `knowledge_base.json` 相关文献
3. 读取前序章节摘要（保证连贯性）
4. 规划段落结构 → 写入文件头部注释
5. 逐段撰写正文

---

### 3. paper_reader — 文献阅读器

**职责**：从 PDF 论文中提取结构化信息，构建知识库。

**提取维度**：
| 字段 | 说明 |
|------|------|
| `core_contribution` | 2-4 句话概括贡献 |
| `method_keywords` | 中英文方法关键词 |
| `dataset` / `metrics` | 数据集和性能指标 |
| `relation_to_our_work` | 四选一：baseline / related / inspired / background |
| `key_numbers` | 关键数字（参数量、推理时间等） |

**约束**：不编造数据，所有指标必须来自文献原文。

---

### 4. ref_manager — 引用管家

**职责**：自动检索文献、生成 BibTeX、补充缺引用的段落。

**工作流程**：
1. 扫描草稿中的 `[PLACEHOLDER: 需文献支撑 — ...]`
2. 生成多维度搜索词（5 种策略：精确匹配、变体、基线、前沿、跨领域）
3. 调用 Semantic Scholar API 检索
4. 相关度打分（标题匹配 30% + 摘要 25% + 引用数 15% + 时效 15% + 发表层级 15%）
5. 输出 BibTeX + 候选文献 JSON

**降级策略**：API 不可用时自动回退到 WebSearch。

---

### 5. draft_viewer — 草稿预览器

**职责**：将英文草稿翻译为中文，生成带 MathJax 渲染的自包含 HTML。

**翻译规范**：
- 意译为主，学术语体
- 人名不翻译，术语首次出现加注英文
- 数学公式原样保留（MathJax 渲染）
- `[TOPIC SENTENCE]` → 黄色高亮块
- `[PLACEHOLDER]` → 红色虚线框

**CSS 设计**：暖白学术配色，820px 居中，衬线字体，响应式布局。

**内置通用术语表**：含 50+ 条跨领域通用学术术语的中英文对照。各项目启动时应追加领域专属术语。

---

## 快速开始

### 第一步：初始化项目

```bash
# 1. 复制本仓库
cp -r paper-writing-framework my-paper-project
cd my-paper-project

# 2. 复制模板文件
cp outline_template.json outline.json
cp knowledge_base_template.json knowledge_base.json
cp session_state_template.json session_state.json

# 3. 编辑 outline.json — 填写你的论文标题、章节、写作要点
#    这是整个项目的蓝图，请认真填写

# 4. 放入参考文献 PDF
#    将你收集的论文 PDF 放入 inputs/references/
```

### 第二步：阅读文献 + 构建知识库

在 Claude Code 中：

```
请使用 paper_reader skill 阅读 inputs/ 下的所有文献，构建 knowledge_base.json。
```

Claude 将逐篇分析 PDF，提取结构化信息并写入知识库。

### 第三步：开始写作

```
请使用 paper_writer skill 写 02_related_work（或你选择的起始章节）。
```

**推荐的写作顺序**：Related Work → Methodology → Experiments → Discussion → Introduction → Conclusion。

原因：Introduction 需要总结全文贡献后精准撰写，适合最后写。

### 第四步：管理引用

当草稿中出现 `[PLACEHOLDER: 需文献支撑 — ...]` 时：

```
请使用 ref_manager skill 为 03_methodology 补充文献。
```

### 第五步：预览进度

```
请使用 draft_viewer skill 生成当前进度的 HTML 预览。
```

### 第六步：恢复进度

在新会话中：

```
请先读取 session_state.json 恢复状态，然后继续写 04_experiments。
```

---

## 适配不同领域

### 计算机科学 / AI / ML

| 调整项 | 说明 |
|--------|------|
| 章节结构 | 标准 6 章即可（Introduction / Related Work / Methodology / Experiments / Discussion / Conclusion） |
| 术语表 | 追加领域术语（如 Backpropagation、Transformer、GPU、CUDA）到 `draft_viewer.md` 的术语对照表 |
| 实验代码 | 放入 `experiments/`，使用 PyTorch / TensorFlow / JAX |
| 图表 | `figures/` 放模型架构图、训练曲线、混淆矩阵等 |

### 工程 / 应用科学

| 调整项 | 说明 |
|--------|------|
| 章节结构 | 可选增加 "System Design" / "Implementation" 章节，修改 `outline.json` 即可 |
| 实验侧重点 | 更关注真实场景测试、鲁棒性、效率指标 |
| 引用来源 | 除论文外，`inputs/patents/` 可放专利 PDF，`relation_to_our_work` 标记为 `baseline` 或 `background` |

### 医学 / 生物信息学

| 调整项 | 说明 |
|--------|------|
| 章节结构 | 可在 Methodology 后增加 "Clinical Validation" 章节 |
| 数据集 | 需详细描述数据集来源、伦理审批、患者数量等 |
| 统计方法 | 实验章节需包含统计分析（p 值、置信区间、效应量） |
| 写作规范 | 可调整 `paper_writer.md` 的写作规范（如时态、术语格式） |

### 社会科学 / 人文

| 调整项 | 说明 |
|--------|------|
| 章节结构 | 可能需要 Literature Review / Theoretical Framework / Case Study 等非标准章节 |
| 语言 | 在 `outline.json` 中设置 `"language": "zh"` 切换为中文学术写作 |
| 引用格式 | 可使用 APA / MLA / Chicago 等，修改 `ref_manager.md` 的 BibTeX 类型映射规则 |
| 实验章节 | 可替换为 "Empirical Analysis" / "Case Study" / "Survey Results" |

### 自定义 Skill

5 个 Skill 文件均为 Markdown 格式的提示词模板，你可以：
- **修改写作规范**：编辑 `skills/paper_writer.md` 中的语言、时态、人称规则
- **调整评分权重**：编辑 `skills/ref_manager.md` 中的相关度打分维度
- **追加术语**：编辑 `skills/draft_viewer.md` 中的术语对照表
- **新增 Skill**：在 `skills/` 下创建新的 `.md` 文件（如 `figure_designer.md`）

---

## 写作规范概要

| 规范 | 默认设置 |
|------|----------|
| 语言 | 学术英文，SCI 期刊标准（可切换为中文） |
| 人称 | **禁止 "we propose"**，用被动语态或 "the proposed method" |
| 引用 | `[REF:Author_Year]` 格式，由 `ref_manager` 统一管理 |
| 数学 | LaTeX 内联 `$...$`，独立公式 `$$...\tag{N}$$` |
| 图表 | `[FIGURE: ...]` / `[TABLE: ...]` 标记 |
| 段落 | 每段以 `[TOPIC SENTENCE]` 开头 |
| 占位 | `[PLACEHOLDER: ...]` 标记待填充内容 |
| 决策 | `[DECISION_NEEDED: ...]` 标记需要人工决策的问题 |

---

## 状态文件说明

### outline.json — 论文蓝图

定义论文的**章节结构**、**写作要点**、**目标字数**、**参考文献覆盖要求**。这是所有 Skill 运行的依据，应在项目启动时仔细填写。

关键字段：`title`、`language`、`total_word_target`、`sections[]`（每章的 `writing_points` 是 paper_writer 的核心输入）、`reference_target`（ref_manager 的覆盖面指引）。

### knowledge_base.json — 文献数据库

`paper_reader` 的输出，`paper_writer` 的输入。每条文献记录核心贡献、方法关键词、数据集、指标、与本文的关系分类。

`relation_to_our_work` 字段控制 paper_writer 如何使用该文献：
- `baseline` → 放入 Experiments 的对比表格
- `related` → 放入 Related Work 正文
- `inspired` → 在 Methodology 中提及但不过度暴露
- `background` → 在 Introduction 中作为背景铺垫

### session_state.json — 跨会话状态

`context_manager` 的核心文件，记录：
- 已完成章节的摘要（100 词）、字数、状态
- 正在写的章节及进度
- 术语表（保证全篇术语一致）
- 引用池（每篇引用出现在哪些章节）
- 决策日志（所有关键决策及理由）
- 上下文快照（防止溢出）

---

## 实验代码整合

`experiments/` 目录用于存放实验代码。建议结构：

```
experiments/
├── run_all.py            # 主实验套件（支持 --tables 1,2,3 --plot）
├── model.py              # 模型定义
├── data_prep.py          # 数据预处理
├── train.py              # 训练脚本
├── evaluate.py           # 评估脚本
├── metrics.py            # 评估指标实现
├── fill_placeholders.py  # 自动将实验结果填充到论文 PLACEHOLDER
├── requirements.txt      # Python 依赖
└── results/              # 实验结果（JSON / CSV）
```

`fill_placeholders.py` 是关键脚本：读取实验结果 JSON/CSV，自动替换论文草稿中的 `[PLACEHOLDER: ...]` 标记。

---

## 常见问题

### Q: 可以不按 6 章结构吗？

完全可以。编辑 `outline.json`，增删 `sections` 数组中的章节即可。所有 Skill 会动态适配。

### Q: 可以写中文论文吗？

可以。在 `outline.json` 中设置 `"language": "zh"`，`paper_writer` 会切换为中文学术写作标准。

### Q: 如何处理 LaTeX 而非 Markdown？

草稿以 Markdown 格式编写，最终可以：
1. 用 Pandoc 转换为 LaTeX：`pandoc final/paper_v1.md -o final/paper.tex`
2. 或在 `tools/` 下添加自定义转换脚本

### Q: 如何与 Overleaf / Word 协作？

- **Overleaf**：将 `references/references.bib` 导入 Overleaf 项目，草稿内容通过 Pandoc 转换为 LaTeX
- **Word**：使用 Pandoc 转换为 `.docx`，或直接在 `final/` 中维护 `.docx` 版本

### Q: 如何添加新的 Skill？

在 `skills/` 目录下创建 `<skill_name>.md`，定义角色、触发条件、输入、输出、执行流程。Skill 之间通过 JSON 状态文件协作。

---

## 与 paper-project 的关系

本框架源自 [paper-project](https://github.com/lanmeinai/paper-project)（RPS-DDF 肝脏点云配准论文写作项目）的完整提炼。paper-project 包含该特定论文的所有领域相关内容（肝脏配准领域术语、3D-IRCADb-01 数据集实验代码、22 篇参考文献 PDF 等），而本框架仅保留**领域无关的写作系统和流程**。

如果你要写的是**医学图像 / 点云配准 / 手术导航**领域的论文，建议直接使用 [paper-project](https://github.com/lanmeinai/paper-project) 作为模板，它包含更多领域特定的实验代码和文献。

如果你要写的是**其他任何领域**的论文，从本框架开始。

---

## 依赖

- **Claude Code**（AI 写作助手）
- **Python 3.10+**（实验代码，可选）
- **Semantic Scholar API**（参考文献检索，免费无需 API Key）
- **MathJax CDN**（HTML 预览中的公式渲染，仅需浏览器）

---

## 许可

MIT License — 自由使用、修改、分发。

---

## 贡献

欢迎提交 Issue 和 PR：

- 新增领域术语到 `draft_viewer.md` 的术语对照表
- 改进 Skill 的执行流程
- 新增学科特定的 outline 模板
- 报告 Bug 或提出功能建议
