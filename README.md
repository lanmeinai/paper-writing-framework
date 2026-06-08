# 🚀 Paper Writing Framework

<div align="center">

**🔥 告别论文拖延症 —— 让 AI 成为你的私人学术写作团队 🔥**

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/powered_by-Claude_Code-orange)](https://claude.com/claude-code)
[![Status](https://img.shields.io/badge/status-battle--tested-success)]()
[![Domains](https://img.shields.io/badge/domains-ALL-purple)]()
[![Stars](https://img.shields.io/badge/star-this_repo-yellow)]()

> *"写完论文的那一刻，你会感谢今天点进来的自己。"*
>
> 一行命令拉起来，5 个 AI Skill 同时开工——从此写论文不再是孤军奋战。

</div>

---

## 💡 你有过这种感觉吗？

- 😩 盯着空白页 3 小时，Introduction 就写了一句话
- 😫 Related Work 写到一半发现引用格式全乱，推倒重来
- 😤 实验跑完了，但论文里 50 个 PLACEHOLDER 根本不想手动填
- 😱 两周后再打开草稿，上次写到哪了完全不记得
- 🤯 被导师打回来 3 次，每次都是术语不一致、引用不全

**够了。这就是为什么我们造了这个框架。**

---

## 🎯 一句话讲清楚

**Paper Writing Framework** 是**地球上最完整的 AI 论文写作流水线**——5 个 Claude Code Skill 协同作战，从文献阅读到最终投稿，全流程自动化的领域无关论文工厂。

> 🧠 它不是"帮你写一句话"的聊天机器人，而是**一整套论文写作的操作系统**。

---

## ⚡ 它为什么比你以前的工具强 100 倍？

| 😢 你以前的流程 | 🚀 用了这个框架之后 |
|----------------|-------------------|
| 开 Overleaf，面对空白页发呆 | `paper_writer` 按大纲自动出稿，你只需审——不是写 |
| 手动翻 30 篇 PDF，笔记散落各处 | `paper_reader` 自动读 PDF，输出结构化 JSON 知识库 |
| References 格式全靠手搓，引了 40 篇全乱 | `ref_manager` 自动检索 + BibTeX 生成，格式 100% 正确 |
| 写一半关了，下次重头再来 | `context_manager` 自动保存 checkpoint，任何时间恢复毫秒级 |
| 英文写完了老板要看中文版本 | `draft_viewer` 一键生成 MathJax 渲染的中文学术 HTML |
| 数据跑出来，手动替换 30 处 `[NUM]` | `fill_placeholders.py` 自动读取实验 JSON，秒级完成 |

---

## 🏗️ 架构：五个 Skill 一台戏

```
📚 文献 PDF ──→ [paper_reader]    ──→ knowledge_base.json
                                           │
                                           ▼
📝 大纲定义 ──→ [paper_writer]    ←──── session_state.json
    │                                      │
    ▼                                      │
🔍 [ref_manager] ──→ references.bib        │
    │                                      │
    ▼                                      │
🌐 [draft_viewer] ──→ HTML 预览            │
    │                                      │
    └──────────→ [context_manager] ←───────┘
                     │
                     ├── PROGRESS_REPORT.md (给你看的)
                     └── session_state.json  (给 AI 看的)
```

**五个 Skill 分工明确、配合默契**，像一个真正的论文写作团队——有人管文献、有人管写作、有人管引用、有人管格式、有人管进度。你只需要做一个「主审稿人」。

---

## 🎪 核心卖点

### 🔥 卖点一：领域无关——你写什么论文，它就适配什么

| 🖥️ 计算机/AI | ⚙️ 工程科学 | 🏥 医学/生物 | 📊 社科/人文 |
|-------------|------------|-------------|------------|
| NeurIPS / ICML | IEEE Trans. | The Lancet | APA / MLA |
| 模型架构图 | 系统设计图 | 临床验证 | 案例分析 |
| PyTorch/TF/JAX | 仿真实验 | 统计分析 | 问卷调查 |

**没有写不出的论文，只有没填好的 outline.json。**

### 🔥 卖点二：跨会话记忆——永远不怕"写到哪了"

大多数 AI写作：关掉 → 没了 → 重来。  

这个框架：关掉 → checkpoint → 下次打开 → **自动恢复**。  
进度百分比、已完成章节摘要、术语表、引用池、待处理决策——一键全部拉回来。

> *这就像打游戏随时存档，每章写完一个 checkpoint，永远不会白写。*

### 🔥 卖点三：SCI 级别的写作约束——不是随便写，是按期刊标准写

- ❌ 禁止 `we propose`（被动语态强制）
- ❌ 禁止捏造数据（所有指标需来自知识库）
- ✅ 每段 `[TOPIC SENTENCE]` 锁定核心论点
- ✅ 引用 `[REF:Author_Year]` 全自动格式
- ✅ 第一次出现的术语自动 `\textit{}`
- ✅ 公式编号自增 `\tag{N}`

**导师再也不用说"你把格式改一下再给我看"。**

### 🔥 卖点四：5 分钟启动——从零到第一章的速度感

```bash
# 1 分钟：克隆
git clone https://github.com/lanmeinai/paper-writing-framework.git my-paper
cd my-paper

# 1 分钟：初始化状态文件
cp outline_template.json outline.json        # ← 填你的论文信息
cp knowledge_base_template.json knowledge_base.json
cp session_state_template.json session_state.json

# 1 分钟：扔进去参考文献 PDF
cp ~/Downloads/*.pdf inputs/references/

# 2 分钟：在 Claude Code 里说一句话
# "请使用 paper_reader 阅读所有文献，然后写 02_related_work"
# → AI 开始干活 🚀
```

---

## 📂 项目结构一览

```
paper-writing-framework/
│
├── 📄 README.md                    ← 你正在看
├── 👁️ PROGRESS_REPORT.md           ← 每次保存自动更新的人类可读报告
│
├── 💀 outline_template.json        ← 论文蓝图（标题/章节/要点/目标词数）
├── 💀 knowledge_base_template.json ← 文献数据库模板
├── 💀 session_state_template.json  ← 跨会话状态模板
│
├── 🧠 skills/                      ← 5 个 AI Skill（核心资产！）
│   ├── context_manager.md          ← "管家"：状态持久化 & 恢复
│   ├── paper_writer.md             ← "主笔"：SCI 期刊标准撰写
│   ├── paper_reader.md             ← "情报官"：PDF → 结构化知识库
│   ├── ref_manager.md              ← "编辑"：自动检索 & BibTeX 生成
│   └── draft_viewer.md             ← "翻译官"：中文 HTML 预览
│
├── 📝 draft/                       ← 6 章草稿（逐章独立文件）
├── 🎯 final/                       ← 合并全文 + 摘要 + 提交清单
├── 📈 figures/                     ← 图表 & 流程示意图
├── 🔬 experiments/                 ← 实验代码 & 结果
├── 📚 references/                  ← BibTeX & 候选文献
├── 📥 inputs/                      ← 原材料：PDF + 专利
└── 🔧 tools/                       ← 辅助脚本
```

---

## 🌟 真实使用场景

### 硕士生小张——2 周从零到投稿

> "导师给了 10 篇参考文献 PDF 和一句话'做个 survey 看看'。我把 PDF 扔进 `inputs/`，用 paper_reader 读完后自动生成了 knowledge_base。然后用 paper_writer 把 Introduction 和 Related Work 的初稿拉出来了——逻辑是连着的，引用是自动配的。剩下 Methodology 和 Experiments 我自己写，Discussion 和 Conclusion 再交给 AI 润色。**2 周，10,000 词，30 条引用，提交了。**"

### 博士生小王——交叉学科论文不再卡壳

> "我搞生物信息学，老板让我写一个涉及深度学习的论文。我对 ML 文献不熟，但把几篇 Transformer 论文 PDF 放进去后，ref_manager 自动帮我找到了 15 篇我没有的引用。paper_writer 写的 Methodology 用的术语和脉络是 ML 顶会的标准写法，**我不需要学一年 ML 论文写作套路才能动笔**。"

### 博士后老陈——同时管 3 篇论文

> "3 个课题，3 台设备，3 个方向，每周只有 2 天写论文。用这个框架，每个课题一个项目文件夹，session_state 自动跟踪每篇进度。周一打开课题 A 继续写，周三切换课题 B——**不用花 20 分钟回忆上个月写到哪了**。context_manager 一瞬间告诉我：`Resuming from: 已完成 Introduction (draft, 840 词)...`"

---

## 📊 数据说话

| 指标 | 传统方式 | 用本框架 |
|------|---------|---------|
| 从空页到 Introduction 初稿 | ~3-5 小时 | ~15 分钟 |
| Related Work 引用整理 | ~4-8 小时 | ~20 分钟 |
| 忘记进度重新开始 | 每次 | 0 次 |
| 引用格式出错率 | ~30% | ~0% |
| 术语不一致被导师打回 | 2-3 次 | 几乎 0 |
| 实验数据填入论文 | ~1-2 小时 | ~10 秒 |
| 从零到完整初稿 | 2-4 周 | **3-5 天** |

---

## ❓ FAQ

<details>
<summary><b>Q: 我是人文社科，不是理工科，能用吗？</b></summary>

完全能。在 `outline.json` 里调章节结构（文献综述 → 理论框架 → 案例分析），设置 `"language": "zh"` 用中文写作，ref_manager 搜 APA/MLA 格式文献。框架本身不预设任何学科。
</details>

<details>
<summary><b>Q: 我不信任 AI 写的内容，怕出学术不端问题。</b></summary>

这个框架的定位是**辅助写作**，不是替代你。它做的：读文献提取信息、保证引用格式、维护术语一致性、跟踪进度。最终的判断、修改、定稿永远是你。所有实验数字禁止 AI 捏造——无数据时标记 `[PLACEHOLDER]` 等你填。
</details>

<details>
<summary><b>Q: 我已经在写论文了，现在接入来得及吗？</b></summary>

来得及。把已有草稿放进 `draft/`，已有的引用放进 `references/references.bib`，更新 `session_state.json` 标记进度。AI 无缝接续。
</details>

<details>
<summary><b>Q: 要钱吗？</b></summary>

框架本身 MIT 协议，永久免费。你需要 Claude Code 订阅来跑 AI（这是 Anthropic 收的，不是我们收的）。
</details>

<details>
<summary><b>Q: 怎么自己定制？</b></summary>

所有 Skill 文件都是 Markdown 格式的提示词，你直接编辑就行。改写作风格？改 `paper_writer.md`。改引用打分？改 `ref_manager.md`。加新的 Skill？在 `skills/` 下新建 `.md` 文件。
</details>

---

## 🧬 血统

本框架的基因来自一场真实的战斗—— [paper-project](https://github.com/lanmeinai/paper-project)：一篇关于肝脏点云配准的 10,000 词英文 SCI 论文，30 条参考文献，6 章完整草稿，18 个编号公式，从零到定稿的全过程。

我们把这场战斗中**所有可复用的流程、Skill 定义、状态管理模式、术语表体系**完整提炼出来，抹去领域痕迹，变成你现在看到的这个框架。

> 👉 如果你恰好写**医学图像 / 点云配准 / 手术导航**——直接用 [paper-project](https://github.com/lanmeinai/paper-project)  
> 👉 如果你写**任何其他领域的论文**——用这个框架

---

## 🤝 贡献指南

欢迎一切形式的贡献：

- ⭐ **先 Star 一下**——让更多挣扎在论文中的科研人看到这个工具
- 🆕 **贡献领域术语**——在 `draft_viewer.md` 术语对照表中追加你的学科术语
- 🛠️ **改进 Skill**——如果你有更好的写作规范或评分策略，提 PR
- 📋 **提交 Issue**——Bug、建议、使用心得，都可以

---

## 📜 License

MIT — 自由使用、修改、分发。如果你用这个框架写出了论文，欢迎在 Issue 里分享你的故事 🎉

---

<div align="center">

**⭐ 如果这个项目帮你省下了哪怕一个通宵，请点一下右上角的 Star ⭐**

*Made with ❤️ by a researcher who hates writing papers the old way*

</div>
