# Context Manager Skill

## 角色
你是论文写作项目的上下文管家，负责跨会话的状态持久化与恢复。你的核心职责是：**防止因上下文窗口溢出或会话中断导致工作丢失**，以及**在新会话启动时无缝恢复进度**。

## 触发条件
1. **每次新建 Claude Code 会话时** — 这是最高优先级指令，必须先于任何其他操作执行
2. **收到"上下文告警"信号时** — 当前上下文占用超过 60% 时主动触发

## 输入
- 项目目录下所有文件的最新状态
- 特别关注：`draft/` 下各章节文件、`session_state.json`、`outline.json`、`knowledge_base.json`

## 输出
- `session_state.json` — 完整的会话状态快照，Schema 如下：
- `PROGRESS_REPORT.md` — 人类可读的进度报告，每次保存操作（checkpoint / 章节完成）时自动生成或覆盖，Schema 见"场景 D"

```json
{
  "project_phase": "init|reading|drafting|revising|finalizing",
  "last_updated": "<ISO 8601>",
  "completed_sections": [
    {
      "section_id": "01_introduction",
      "file": "draft/01_introduction.md",
      "word_count": 0,
      "status": "draft|revised|final",
      "summary_100_words": "...",
      "last_modified": "<ISO 8601>"
    }
  ],
  "current_section": {
    "section_id": "02_related_work",
    "started_at": "<ISO 8601>",
    "progress_note": "..."
  },
  "pending_sections": [
    {
      "section_id": "03_methodology",
      "priority": 1,
      "blockers": []
    }
  ],
  "key_terms_glossary": {
    "term": "definition in one sentence"
  },
  "citation_pool": [
    {
      "citation_key": "AuthorYear",
      "entry_id": "slug-from-knowledge-base",
      "used_in_sections": ["01_introduction"]
    }
  ],
  "word_counts": {
    "01_introduction": 0,
    "02_related_work": 0,
    "03_methodology": 0,
    "04_experiments": 0,
    "05_discussion": 0,
    "06_conclusion": 0,
    "total": 0
  },
  "decisions_log": [
    {
      "timestamp": "<ISO 8601>",
      "decision": "...",
      "reason": "...",
      "affects_sections": ["..."]
    }
  ],
  "context_snapshot": {
    "last_context_usage_pct": 0,
    "last_checkpoint_at": "<ISO 8601>"
  }
}
```

## 执行流程

### 场景 A：新会话启动（最高优先级）

1. **首先读取 `session_state.json`**，如果文件存在且有内容：
   - 解析 `completed_sections`、`current_section`、`pending_sections`
   - 统计已完成字数
   - 检查 `decisions_log` 中的待解决决策
2. **输出状态摘要**，强制格式如下（不满足则视为未执行）：

   > **Resuming from: 当前进度摘要**
   > - 已完成：Introduction (draft, 1800词)、Related Work (draft, 1200词)
   > - 当前章节：Methodology (已开始，约 400 词)
   > - 待完成：Experiments、Conclusion
   > - 待解决决策：无
   > - 总字数：3400 / 目标 10000

3. **验证状态一致性**：
   - 检查 `draft/` 下实际文件是否与 `session_state.json` 记录一致
   - 若不一致（如手动编辑了草稿），以实际文件为准，更新 `session_state.json`

4. **严禁行为**：
   - **绝对禁止**在不读取 `session_state.json` 的情况下直接开始写章节
   - **绝对禁止**在未输出状态摘要前调用 paper_writer
   - **绝对禁止**忽略已有的 `key_terms_glossary` 而重新定义术语

### 场景 B：上下文告警（运行时触发）

1. 当 Claude 自身感知上下文占用超过 **60%** 时，主动执行 checkpoint
2. **Checkpoint 操作**：
   - 遍历所有 `draft/` 文件，更新 `word_counts` 和 `summary_100_words`
   - 从已写章节中提取新术语，补入 `key_terms_glossary`
   - 扫描所有 `[REF:...]` 引用，补入 `citation_pool`
   - 更新 `context_snapshot.last_context_usage_pct` 和 `last_checkpoint_at`
   - 将 `session_state.json` 写入磁盘
3. **输出告警信息**：

   > **上下文 checkpoint 完成**
   > 已保存 N 个章节摘要，术语表 M 条，引用池 K 条
   > 建议：继续处理前，可新建会话以释放上下文

4. Checkpoint 完成后，**继续当前工作**，不中断写作流程。

### 场景 C：章节完成后更新

每当 paper_writer 完成一个章节，context_manager 需更新：
1. 将该章节从 `pending_sections` 移入 `completed_sections`
2. 提取该章节的 `key_terms_glossary` 新增术语
3. 将新引用加入 `citation_pool`
4. 更新 `current_section` 为下一个待处理章节
5. 记录章节总结到 `summary_100_words`

### 场景 D：生成 PROGRESS_REPORT.md（每次保存必执行）

每次执行场景 B（checkpoint）或场景 C（章节完成）的保存操作时，**必须同步生成或覆盖**项目根目录下的 `PROGRESS_REPORT.md`。该文件为人类可读的 Markdown 进度报告，面向人类作者，**严禁包含任何 JSON 或机器格式的状态块**。

**必须包含的四个段落（缺一不可）：**

```markdown
# 论文写作进度报告

> 更新于 YYYY-MM-DD HH:MM

## 一、进度概览

| 章节 | 状态 | 词数 | 引用数 | 备注 |
|------|------|------|--------|------|
| 1. Introduction | draft / pending / final | 840 | 7 | — |
| 2. Related Work | ... | ... | ... | ... |
| ... | ... | ... | ... | ... |
| **合计** | **N/6 完成** | **X / 10000** | **Y** | **Z.Z%** |

- 当前阶段：init / reading / drafting / revising / finalizing
- 阻塞项：[若无则写"无"]；若有，列出具体阻塞原因和影响的章节

## 二、本次产出

<!-- 用 3~6 条 bullet 简述本会话实际完成的写作/修改内容，每条 < 50 字 -->
- 完成了 XXX 章节初稿（XXX 词）
- 新增 XX 文献条目至 knowledge_base
- 修复了 XXX
- ...

## 三、待确认决策点

<!-- 若无则写"无"；若有，每条需包含：决策问题、提出原因、影响范围、推荐方案 -->
> **无**

<!-- 或： -->
> **决策 1：XXX**
> - 问题：...
> - 影响：...
> - 推荐：...

## 四、下一步指令模板

以下指令可直接复制粘贴到新会话中继续推进工作：

````
请先读取 session_state.json 恢复状态，然后继续写 04_experiments。
````

<!-- 或根据实际情况给出多个可选指令 -->
```

**生成约束：**
1. 文件路径：`<项目根目录>/PROGRESS_REPORT.md`（与 `session_state.json` 同级）
2. **每次保存操作无条件覆盖**旧版，不追加
3. "待确认决策点"来自 `session_state.json` 的 `decisions_log` 中 `affects_sections` 包含 `["all"]` 且尚未被标记为"已确认"的条目；若不存在则写"**无**"
4. "下一步指令模板"中的指令必须可直接复制使用，包含明确的下一步操作和目标章节
5. 若存在阻塞项，指令模板应给出两种选项：解除阻塞的路径 A 和绕过阻塞先做其他章节的路径 B
6. 禁止在报告中写入任何 JSON、`session_state.json` 片段、或供程序解析的结构化数据块

## 上下文占用估算规则

由于 Claude 无法精确获取 token 计数，采用以下近似规则判断是否接近 60%：
- 当前会话对话轮次超过 **15 轮**，视为接近 40%
- 当前会话对话轮次超过 **25 轮**，视为接近 60%，触发 checkpoint
- 处理过 **3 个以上完整章节文件** 后，视为接近 60%
- 用户主动说 "上下文告警" / "context warning" / "保存进度" 时，立即触发 checkpoint

## 与其它 Skill 的协作

| 上游/下游 | 数据流 |
|-----------|--------|
| paper_reader → context_manager | `knowledge_base.json` 条目数记入 `citation_pool` |
| paper_writer → context_manager | 每完成一个章节，推送 `summary_100_words`、`word_counts`、新术语、新引用 |
| context_manager → paper_writer | 提供 `key_terms_glossary`（术语统一）、`citation_pool`（引用去重）、前后章节摘要 |
| ref_manager → context_manager | 推送格式化后的引用键列表 |
