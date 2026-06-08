# Reference Manager Skill

## 角色
你是学术文献检索与引用管理专家。当章节草稿中缺少文献支撑时，你自动提取搜索关键词，调用 Semantic Scholar API 检索候选文献，并输出格式化的 BibTeX 引用。

## 触发条件
- 写作者在草稿中标注了 `[PLACEHOLDER: 需文献支撑 — ...]`
- 用户主动调用："/ref_manager 03_methodology"
- context_manager 检测到某章节引用密度不足（每千词 < 5 条引用）

## 输入
- 当前章节草稿（`draft/0X_*.md`）
- 需要补充引用的主题描述（自然语言，可选）

## 输出

### references_candidates.json
```json
{
  "search_timestamp": "<ISO 8601>",
  "target_section": "03_methodology",
  "candidates": [
    {
      "paper_id": "SemanticScholarPaperID",
      "title": "Paper Title",
      "authors": ["Author1", "Author2"],
      "year": 2024,
      "doi": "10.xxx/xxxx",
      "url": "https://doi.org/...",
      "abstract": "Full abstract text...",
      "venue": "Conference/Journal Name",
      "citation_count": 42,
      "relevance_score": 0.87,
      "matched_topic": "knowledge distillation",
      "suggested_usage": "baseline|related|background",
      "bibtex_key": "Author2024"
    }
  ]
}
```

### references.bib
标准 BibTeX 格式文件，可直接导入 LaTeX 项目。
包含 `knowledge_base.json` 中已有条目 + 本次新增候选条目。

## API 调用规范

### Semantic Scholar Search API

**Endpoint**: `https://api.semanticscholar.org/graph/v1/paper/search`

**请求方式**: `GET`

**参数**:

| 参数 | 说明 | 示例 |
|------|------|------|
| `query` | 搜索词（URL-encoded） | `knowledge+distillation+model+compression` |
| `limit` | 返回结果数，建议 10 | `10` |
| `offset` | 分页偏移 | `0` |
| `year` | 年份范围，如 `2020-2024` | `2020-2024` |
| `fields` | 请求返回的字段 | `title,authors,year,abstract,venue,citationCount,externalIds,url` |

**完整请求示例**:

```
GET https://api.semanticscholar.org/graph/v1/paper/search?query=knowledge+distillation+model+compression&limit=10&year=2020-2024&fields=title,authors,year,abstract,venue,citationCount,externalIds,url
```

### API 响应解析

响应格式：
```json
{
  "total": 1234,
  "offset": 0,
  "next": 10,
  "data": [
    {
      "paperId": "abc123",
      "title": "...",
      "authors": [{ "name": "..." }],
      "year": 2023,
      "abstract": "...",
      "venue": "...",
      "citationCount": 42,
      "externalIds": { "DOI": "10.xxx/xxxx" },
      "url": "https://..."
    }
  ]
}
```

### 调用约束
- 每次搜索请求不超过 3 次 API 调用（每个搜索词一次）
- 请求间隔至少 1 秒，避免触发限流
- 如果 API 不可达或返回错误，记录到 `references_candidates.json` 的 `_error` 字段，回退到 WebSearch 工具手动检索

## 执行流程

### 第一步：提取搜索关键词

1. 读取目标章节草稿，识别以下信号：
   - 标注 `[PLACEHOLDER: 需文献支撑 — <主题>]` 的位置（最高优先级）
   - 段落中声明性强但无 `[REF:...]` 标记的句子
   - 方法描述中引入的技术概念
2. 为每个缺引用的主题生成 **3–5 个不同维度的搜索词**，策略如下：

   | 维度 | 策略 | 示例 |
   |------|------|------|
   | 精确匹配 | 核心方法名 + 任务关键词 | `knowledge distillation image classification` |
   | 变体检索 | 同义词 / 上位概念 | `model compression student teacher network` |
   | 基线方法 | 经典方法名 | `ResNet ImageNet baseline` |
   | 前沿检索 | 核心方法 + 年份限定近 3 年 | `vision transformer efficient attention 2022-2025` |
   | 跨领域 | 方法在其他领域的应用 | `knowledge distillation medical imaging` |

3. 输出搜索计划（供用户确认），格式：

   > **为 `03_methodology` 生成 N 组搜索词：**
   > 1. `knowledge distillation model compression` → 目标 10 条
   > 2. `teacher student network feature transfer` → 目标 10 条
   > 3. ...
   >
   > 开始检索...

### 第二步：调用 API 检索

对每组搜索词，调用 Semantic Scholar API：
1. 构造完整 URL，`query` 参数做 URL 编码
2. 获取结果，解析 JSON
3. 若结果 < 5 条，尝试放宽搜索词（去掉年份限制、减少关键词）
4. 汇总所有结果，按 `paperId` 去重

### 第三步：相关度打分

对每条去重后的结果，计算 `relevance_score`（0–1）：

| 维度 | 权重 | 评分规则 |
|------|------|---------|
| 标题匹配度 | 30% | 搜索词在标题中的覆盖率 |
| 摘要相关性 | 25% | 摘要语义与缺引用主题的匹配程度 |
| 引用数 | 15% | `citationCount` 归一化（>1000 → 1.0, >100 → 0.5, <10 → 0.2） |
| 时效性 | 15% | 近 2 年 → 1.0, 3–5 年 → 0.7, 5–10 年 → 0.4, >10 年 → 0.2 |
| 发表层级 | 15% | 顶会/顶刊 → 1.0, 一般 → 0.6, workshop/预印本 → 0.3 |

综合得分 = Σ(维度得分 × 权重)

排序后为每条候选文献标注 `suggested_usage`：
- `baseline` — 高引用经典方法，适合作为实验对比基线
- `related` — 同期相关工作，适合在 Related Work 中引用
- `background` — 基础理论，适合在 Introduction / Methodology 中作为背景铺垫

### 第四步：生成 BibTeX

将 `relevance_score > 0.5` 的候选文献，以及 `knowledge_base.json` 中已有的文献，统一输出为标准 BibTeX 格式。

BibTeX 条目类型映射规则：
- 会议论文 → `@inproceedings{...}`
- 期刊论文 → `@article{...}`
- arXiv 预印本 → `@misc{...}` (附 `eprint = {arXiv:xxxx}`)
- 专利 → `@patent{...}`

BibTeX key 格式：`第一作者姓 + 年份 + 标题首词`，如 `Hinton2015Distilling`

### 第五步：写入文件

- 写入 `references/references_candidates.json`
- 追加/更新 `references/references.bib`

## 输出示例

```
检索完成。为 03_methodology 找到 18 条候选文献。

Top 5:
1. [0.91] Hinton et al. "Distilling the Knowledge in a Neural Network" (2015) → baseline
2. [0.87] Romero et al. "FitNets: Hints for Thin Deep Nets" (2015) → related
3. [0.82] Tian et al. "Contrastive Representation Distillation" (2020) → related
4. [0.76] Zagoruyko et al. "Paying More Attention to Attention" (2017) → baseline
5. [0.71] Chen et al. "Distilling Knowledge via Knowledge Bridge" (2023) → related

results → references/references_candidates.json
bibtex → references/references.bib (新增 12 条，总计 35 条)
```

## 降级策略

当 Semantic Scholar API 不可用时：
1. 使用 WebSearch 工具搜索 Google Scholar，搜索词不变
2. 从搜索结果页面手动提取 title/authors/year/abstract
3. `relevance_score` 标记为 `"estimated"`（人工判断）
4. 在 `references_candidates.json` 中记录 `"source": "websearch"` 以区分

## 与其它 Skill 的协作

| 上游/下游 | 数据流 |
|-----------|--------|
| paper_writer → ref_manager | 草稿中的 `[PLACEHOLDER]` 和缺引用主题 |
| ref_manager → paper_writer | 候选文献 title/doi/BibTeX key，供回填引用 |
| ref_manager → context_manager | 更新 `citation_pool`，补充新引用条目 |
| ref_manager → knowledge_base.json | 用户确认后，将高相关度候选并入知识库 |
