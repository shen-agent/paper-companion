# 文献综述生成 (W6)

输出 manuscript-style 文献综述：20-50 篇文献、主题导向、引用经过验证、显式压制 AI 味儿。区别于 W2 综合笔记（私人 / 30 分钟 / 3-10 篇），W6 产出可作为发表草稿或论文前置 chapter。

> **Agent 参数引用**：本文中 `{citation_style}` / `{note_language}` / `{review_length}` 来自 `agents/paper-companion-agent/agent.yaml`。默认值：`citation_style=informal`、`note_language=zh`、`review_length=medium`。

> **不在作用域**：PRISMA-grade 系统综述（500+ 摘要筛选、多 reviewer、双盲、质量评估工具）。如需 SLR，本工作流可作为起点但不负责完整流程——后续可拓展 W7。

## 工作流（7 步）

### Step 1 — 范围定义

询问用户：
- **主题**（一句话陈述）
- **核心问题**（综述要回答什么；3-5 个为佳）
- **文献来源**：Zotero collection / W3 库内查询 / W5 外源搜索 / 显式 itemKey 列表
- **时间窗**：默认近 5 年；用户可调
- **目标长度**：`short` 3-5 页 / `medium` 6-10 页 / `long` 10-15 页（来自 `{review_length}`）
- **citation_style**：默认 `{citation_style}`，可现场切换
- **是否走 PICO 框架**：临床/生物医学场景推荐（Population / Intervention / Comparison / Outcome）

输出：scope 摘要 → confirm。

### Step 2 — 文献圈定

按来源调用 W3 / W5 / `get_collection_items`：
- 输出候选清单（标题 + 作者 + 年份 + 来源），按相关度排序
- **20-50 篇是 sweet spot**：<10 太薄；>100 建议走 SLR
- 用户 confirm：全部纳入 / 排除哪几篇 / 增加哪些

输出：最终文献清单 + matrix（Excel-style 表格预览） → confirm。

### Step 3 — 风格校准

**写之前必问**："你能给我 1-3 篇过往论文/综述作为风格参考吗？路径或 itemKey 都行。"

- 用户给：`get_content` 读样本 → 提取
  - 段落开头模式（claim-first / question-first / observation-first）
  - 句长分布（短句/长句比例）
  - 用词偏好（被动 vs 主动；具体术语）
  - 章节标题习惯
- 用户不给：用领域中性学术风格 + 强力反 AI 味儿过滤
- **优先级**：领域规范（硬）> 期刊规范（强）> 个人偏好（软）

### Step 4 — 主题归类

逐篇阅读：
- 优先复用已有 `auto:deep-read` note（W1 产物）→ 省 token
- 否则 `get_content(mode="complete")` 读全文
- 边读边按 **3-5 个 themes** 归类（>5 用户认知负担过重；<3 综述偏薄）

输出 theme matrix：

| Theme | 文献 | 关键 claim | 方法 | 我的理解 |
|-------|------|-----------|------|---------|
| T1 X | A,B,C | … | … | … |
| T2 Y | D,E | … | … | … |

→ confirm theme 划分。

### Step 5 — 大纲设计

按 `{review_length}` 控制规模：

```
Abstract（≤250 词）
1. 引言
   1.1 主题动机（具体数字 / 临床痛点 / 方法瓶颈，不写"近年来发展迅速"）
   1.2 综述边界（纳入/排除标准）
   1.3 综述结构概览
2. Theme 1
3. Theme 2
...
N. 跨主题综合（emerging 共识 / 持久分歧 / 方法学缺口）
N+1. 缺口与未来方向（≥3 个 specific 项）
N+2. 结论
References
```

输出大纲 → confirm。

### Step 6 — 草稿生成 + 反 AI 味儿过滤

按节写。每节生成后**立即扫黑名单**（见下方"反 AI 味儿规范"），命中即重写。

每节遵循：
1. **具体观点**优先（不是文献堆砌）
2. **引用具体到数字 / 图表**（"Liu (2024) 报告 AUC=0.89 (Fig 2a)"，不是"Liu 取得了显著效果"）
3. **批判**（每个 theme 至少一处 ⚠ 显式标注分歧 / 局限）
4. **引用按 `{citation_style}` 渲染** + 每条 cite 验证 DOI

输出完整草稿 → confirm。

### Step 7 — 输出

写入位置：
- **默认**：本地 markdown `lit-review_<topic-slug>_<YYYYMMDD>.md`
- **可选**：Zotero standalone note（写入位置说明 + 链接到本地路径）
- **可选**：PDF（如果 pandoc 可用）

写完返回路径 + 字数统计 + 引用条数。

## 反 AI 味儿规范

### 黑名单（出现即改写）

**英文（25 + 词）**：

| 类别 | 词/短语 |
|------|--------|
| 时间套话 | "In recent years"、"With the rapid development of"、"Over the past decade"（开头慎用，必须接具体数字）|
| 知识断言 | "It is widely known/recognized/believed/accepted"、"It has been demonstrated"（被动） |
| 重要性形容 | "plays a crucial/critical/important/significant role"、"of paramount importance"、"of great significance" |
| 量级模糊 | "various"、"numerous"、"several"（无数字）、"many"、"a multitude of" |
| 价值断言 | "robust"、"comprehensive understanding"、"in-depth analysis"、"thorough investigation" |
| 营销腔 | "delve into"、"shed light on"、"open up new avenues"、"holds promise"、"paradigm shift" |
| 学术套词 | "cutting-edge"、"state-of-the-art"、"novel"（无具体新颖点）、"groundbreaking" |
| 连接词滥用 | "Furthermore"、"Moreover"、"Additionally"（每段都用）、"In addition"、"Notably" |

**中文**：

| 类别 | 词/短语 |
|------|--------|
| 时间套话 | "近年来"、"随着 X 的快速发展"、"伴随着..."（开头慎用，必须接具体数字）|
| 知识断言 | "众所周知"、"广泛认为"、"普遍接受" |
| 重要性形容 | "具有重要意义"、"具有重要价值"、"举足轻重"、"至关重要" |
| 营销腔 | "为...提供了新思路 / 新方向 / 新视角"、"开辟了新天地"、"前景广阔" |
| 关注度套话 | "蓬勃发展"、"广受关注"、"备受瞩目"、"日益受到重视" |
| 综合套话 | "综合而言"、"总的来说"（结尾每节都用）|

### 结构规则

- **em dash 一节 ≤ 3 个**：超过即拆分或换为冒号/破折号外的标点
- **段落开头多样化**：不要每段都 `Smith et al. (2023) found that...`
  - 可用开头：claim、question、contrast、observation、definition、mechanism description
  - 同一节内同样开头模式 ≤ 2 次
- **句长变化**：不全是 25-30 词均匀长句；穿插 8-15 词短句强化节奏
- **避免三连排比**：长期读会形成 AI 节拍感（"X, Y, and Z" 模式不能段段都有）

### 抽象 → 具体 替换

| AI 味儿（抽象）| 改成（具体）|
|---------------|------------|
| "Several studies showed X" | "Liu (2024) 和 Chen (2023) 报告 X；但 Watkins (2022) 在 N=350 cohort 中观察到反向（HR=1.2 vs 0.42）" |
| "Significantly improved performance" | "AUC 从 0.78 提升到 0.89（p<0.001）" |
| "Various methods have been proposed" | 直接列：fragmentomics（Liu）、end-motif（Chen）、size-based（Watkins）|
| "It has been demonstrated that X" | "Watkins (2022) 在 350 例 cohort 中证明 X" |
| "These findings suggest" | "这意味着 …（具体推论）"|
| "Furthermore / Moreover" 滥用 | 改成具体逻辑词："In contrast"、"By comparison"、"Crucially"、"Counterintuitively" |
| "Shows promise" | 给具体应用场景 + 当前性能数字 |
| "近年来研究表明" | "Liu (2024) 和后续 3 篇工作表明..." |

## 引用验证（硬门槛）

实测显示 LLM 即使经 3 轮校验仍有 **31% 引用错误率**（捏造作者、错误年份、不存在的 DOI、错误期刊）。

**强制规则**：
1. **每条 cite 都验证**：优先 `mcp__paper-search-mcp__get_crossref_paper_by_doi(doi=...)`，次选与 Zotero `get_item_details` metadata 比对
2. **不允许凭"记忆"引用**：训练数据里看过的不算，必须查
3. **DOI 不可用时**：标 `[DOI 未验证]`，不要伪造
4. **作者列表 / 年份 / 期刊不一致时**：以 CrossRef 权威 metadata 为准，覆盖 Zotero
5. **预印本特殊处理**：标 `[Preprint]` 或 `[bioRxiv]`，避免误以为已正式发表

## 三层任务委派

| 层 | 任务 | agent 行为 |
|----|------|-----------|
| **Tier 1 自动** | 文献清单、引用渲染、字数统计、模板填充 | 直接做 |
| **Tier 2 协作** | 主题归类、草稿生成、theme matrix、反 AI 过滤 | 给草稿 → user confirm |
| **Tier 3 苏格拉底** | 缺口识别、理论框架、新颖性判断、跨 theme 综合 | 主动反问、不替用户拍板（如"你认为 T1 和 T3 的张力点是什么？"）|

## 三大失败模式 + 缓解

| 失败 | 表现 | 缓解 |
|------|------|------|
| **引用幻觉** | LLM 生成像真的但不存在的 paper | 强制 CrossRef 验证；不许凭记忆 |
| **上下文剥离** | 摘录 claim 时丢失"in cohort X" 等关键限定 | 引用关键 claim 必带页码 / Fig 编号 / cohort 描述 |
| **虚假共识** | 把不同立场强行说成"互补" | 每个 theme 必须显式找出 ≥1 处分歧；找不到说明读得不深，回去补 |

## 综述模板

```markdown
# {主题}: A Narrative Review

**Authors**: {User Name}
**Date**: {YYYY-MM-DD}
**Generated with**: paper-companion W6 (Claude); 引用经 CrossRef 验证

## Abstract

**Background**. {一句话痛点。具体数字。}
**Objective**. 本综述系统梳理 X 方向的 N 篇工作，聚焦 {3 个 themes}。
**Methods**. 文献来源 {Zotero collection / 数据库 / 时间窗 / 纳入标准}。
**Findings**. {三条最关键发现，每条带具体数据}。
**Conclusion**. {一句话推论 + 最大未解决问题}。

## 1. Introduction

### 1.1 主题动机
（具体痛点：X 病发病率 Y%；当前金标准的具体瓶颈 Z；多少病人受影响）

### 1.2 综述边界
- 纳入：{具体标准}
- 排除：{具体标准}
- 时间窗：{YYYY-YYYY}
- 检索来源：{databases}

### 1.3 综述结构
本文按 {N} 个 themes 组织：T1 ...; T2 ...; T3 ...

## 2. Theme 1: {主题名}

### 2.1 主题定义
（一句话精确定义 + 为什么重要 + 与 T2/T3 的边界）

### 2.2 关键研究综述
（thematic 而非 study-by-study；同主题相关的 paper 一起讨论；用具体数字）

### 2.3 方法学比较
| 文献 | 数据规模 | 关键技术 | 对照 | 关键结论 |
|------|---------|---------|------|---------|
| ... | ... | ... | ... | ... |

### 2.4 跨研究 synthesis
- 共识点（≥2 篇支持）：…
- ⚠ 分歧点：A 主张 X；B 主张 Y。冲突源于 {cohort / 方法 / 时机}
- 该主题的未解决问题

## 3. Theme 2: ...
（同上）

## N. 跨主题综合

- **Emerging 共识**：{跨 theme 的趋同结论}
- **持久分歧**：{跨 theme 的不一致}
- **方法学缺口**：{什么没人做 / 什么样本类型没覆盖}

## N+1. 缺口与未来方向

至少 3 个 **specific** 项：
1. **{Gap 1}**：{具体到样本类型 / cohort / 技术 / 验证方法}。建议路径：…
2. **{Gap 2}**：…
3. **{Gap 3}**：…

## N+2. Conclusion

（≤200 词。一句话核心结论 + 1-2 句最关键开放问题）

## References

按 `{citation_style}` 渲染。每条经 CrossRef 验证（CrossRef 不可达的标 `[DOI 未验证]`）。
```

## 5 维质量检查清单（Step 6 后必跑）

- [ ] **引用完整性**：每条 cite 都过 DOI 验证；缺 DOI 的标记 `[未验证]`
- [ ] **主题导向**：检查至少 70% 内容是 thematic synthesis 而非 study-by-study summary
- [ ] **反 AI 味儿**：扫黑名单 0 命中；段落开头多样化；em dash ≤3/节
- [ ] **批判性**：每个 theme ≥1 处 ⚠ 分歧标注
- [ ] **缺口具体**：未来方向 ≥3 项 + 每项给具体路径建议（不是"未来还有很多工作"）

## 完整范例（虚构主题）

下方是 ctDNA 早癌检测综述的**第 1 个 theme 段落**——展示反 AI 味儿+具体引用+批判性的实际效果：

```markdown
## 2. Theme 1: 片段化模式 (Fragmentomics) 作为 HRD signature

cfDNA 片段长度分布在肿瘤来源样本中系统性偏离正常背景：肿瘤来源片段中位长度
比正常 cfDNA 短 8-15 bp（Watkins, 2022, *Nat Med*, N=350）。这个观察构成了
fragmentomics-based HRD 检测的物理基础。

Liu (2024) 在 BRCA 突变型卵巢癌（N=120）中将片段长度与末端基序谱（end motif）
组合成 logistic 分类器，验证集 AUC 达 0.89——明显超过单一信号方案
（Chen, 2023, *Genome Med*, N=200, AUC=0.84）。两项工作在高肿瘤分数（TF>5%）
样本上结论一致。

⚠ **分歧出现在低 TF 子集**。Liu 报告 TF<3% 的样本灵敏度仍达 78%，而 Chen 在
同区间报告 45%。差异可能源于三处：cohort 阶段（复发 vs 一线，TF 中位 8% vs 2%）、
panel 深度（5× vs 2×）、以及末端基序数量（128 vs 64）。这意味着 fragmentomics
方法的外部有效性高度依赖样本组成——单一 cohort 的灵敏度数字不可外推。

方法学上，三项工作都使用 logistic regression 而非更复杂的深度模型。Tanaka (2025,
*Cell Rep Med*) 尝试 transformer 架构但仅小幅提升（AUC 0.91 vs 0.89），代价是
可解释性下降——这一权衡值得后续工作明确评估。

**该主题未解决的问题**：(i) 低 TF 增敏的最佳 panel 设计参数尚无共识；(ii) 非
BRCA 突变 HRD 表型（如 BRCA-like）的 fragmentomics 表现仅 Tanaka 一篇验证，
N=180 不足以推论；(iii) 多中心一致性数据缺失。
```

注意上文：
- 0 个黑名单词
- 每个 claim 都有具体数字 + 文献 + 图表 / 章节
- 显式 ⚠ 标分歧并解释 3 个可能原因
- 段落开头多样：观察 → 数据点 → 分歧 → 方法学比较 → 开放问题
- 自然中英文混排（不强制单一语言）
- 句长有变化

## 错误处理

| 情况 | 处理 |
|------|------|
| 文献 < 10 篇 | 警告"综述偏薄"；建议先扩搜（W5）或改走 W2 综合笔记 |
| 文献 > 100 篇 | 建议走 SLR；W6 不适合；如坚持，让用户先按子主题切分 |
| 用户没给风格样本 | 用领域中性 + 强黑名单过滤；输出顶部说明"未做 voice 校准" |
| CrossRef 不可达 | 该条引用标 `[DOI 未验证]`；不阻塞流程，但在质检报告里列出 |
| 用户要求 SLR-grade | 明告 W6 不支持；列 PRISMA 需要的额外步骤；建议未来 W7 |
| 草稿过长 / 过短 | 与 `{review_length}` 目标偏差 >30% 时主动告知，等用户决定是否调整 |

## 不要做的事

- ❌ 不要 study-by-study 写（"Smith 2020 发现 ... Jones 2021 发现 ... Davis 2022 发现 ..."）
- ❌ 不要凭训练数据"记得"引用——必须查
- ❌ 不要为了让叙述顺滑而抹平真实分歧
- ❌ 不要每段都 "Furthermore" 起头
- ❌ 不要在没拿到 user confirm 前直接写 7000 字草稿
- ❌ 不要伪造 metadata 字段（缺 DOI 就标缺，不要编）
- ❌ 不要把 W6 输出包装成"已发表论文"——它是草稿，需要用户深度修订
