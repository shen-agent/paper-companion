# 文献 Wiki 维护 (W8)

把 paper-companion 的 W1/W2/W6 等工作流的产出沉淀为**带 wikilink、可查询、可健康审计**的 Karpathy-style LLM Wiki。区别于 Zotero note（埋在 child note 里），wiki 是跨会话累积的知识图谱，**最大价值在 concept page**——同一概念在 N 篇文献里的综合视图。

> **Agent 参数引用**：本文中 `{wiki_subdir}` / `{wiki_use_bases}` 来自 `agents/paper-companion-agent/agent.yaml`。默认值：`wiki_subdir=llm-wiki/lit-wiki`、`wiki_use_bases=true`。

> **作用域硬边界**：W8 操作严格限制在 `{wiki_subdir}/` 范围内（通过 obsidian-cli 在 vault 内操作）。**不修改** vault 内其他目录（如 `40-文献阅读/`、`98-模板/`）。如发现需要操作其他目录，停止并询问用户。

> **存储职责划分**：
> - **Zotero**（已有）：raw source 层——PDF、annotations、metadata
> - **Obsidian lit-wiki**（W8）：distillation 层——entity pages、cross-links、health audits

---

## 🔴 CRITICAL：操作 Obsidian 前必读 skill

**任何 vault 写入或读取操作前**，必须先 `Skill` 工具加载对应 obsidian skill：

| 即将操作 | 必读 skill（按顺序）|
|---------|------------------|
| 任意 vault 写入（create/append/property:set）| 1️⃣ `obsidian-cli` 2️⃣ `obsidian-markdown` |
| 创建 / 修改 `.base` 文件 | + 3️⃣ `obsidian-bases` |
| 仅读取 vault 内容（read/search/backlinks）| `obsidian-cli` |

**Why**：
- `obsidian-cli` 是执行通道，命令语法在那里
- `obsidian-markdown` 是 Obsidian Flavored Markdown 语法权威（wikilink、callout、frontmatter properties、embeds）
- `obsidian-bases` 是 `.base` 文件 schema 与 50+ 函数

**永远走 obsidian-cli**，不要直接用 Write/Edit 工具写 vault 路径——避免与运行中的 Obsidian 客户端写冲突（写完 obsidian 客户端可能 silently 覆盖）。

**自检**：每次会话 W8 首次操作前，确认上述 skill 已 load。如未 load 直接调 `obsidian` CLI 是不行的——必须先经 Skill 工具。

---

## 5 个子工作流速查

| ID | 名称 | 触发关键词 |
|----|------|----------|
| **W8a** | wiki-init | "初始化 lit-wiki"、"建一个文献 wiki"、"setup wiki" |
| **W8b** | wiki-ingest | "ingest 这篇到 wiki"、"加进 wiki"、"从这篇文献更新 wiki" |
| **W8c** | wiki-query | "wiki 里有没有 X"、"在 wiki 里查 X"、"X 在 wiki 里怎么说" |
| **W8d** | wiki-lint | "检查 wiki 健康"、"lint wiki"、"wiki audit" |
| **W8e** | wiki-update | "改 wiki 里的 X"、"修订 X 这一页"、"更新 X 的 confidence" |

---

## W8a wiki-init（一次性 bootstrap）

### 触发与确认

询问用户：
- vault 路径（默认 `{obsidian_vault_path}`）
- wiki 子目录（默认 `{wiki_subdir}`）
- 是否生成 `.base` 数据视图（默认 `{wiki_use_bases}`）
- 主题域（如"基因组学 / cfDNA / 早癌检测"）——会进 README

输出 init 计划 → confirm。

### 创建骨架（共 10-14 个文件）

通过 `obsidian create` 依次建：

| 路径 | 类型 | 内容来源 |
|------|------|---------|
| `{wiki_subdir}/SCHEMA.md` | 维护守则 | 复制 `wiki-schema-template.md` 内容 |
| `{wiki_subdir}/README.md` | 给人读的说明 | 模板 + 用户主题域 |
| `{wiki_subdir}/index.md` | 主目录 | 模板（按 type 分组：papers/concepts/methods/people/datasets）|
| `{wiki_subdir}/log.md` | 操作日志 | 单行：`[YYYY-MM-DD] init` |
| `{wiki_subdir}/overview.md` | 顶层综述 | 占位（"待 ingest 后增量填充"）|
| `{wiki_subdir}/glossary.md` | 术语表 | 占位 |
| `{wiki_subdir}/pages/{papers,concepts,methods,people,datasets}/.gitkeep` | 5 个空目录占位 | — |

如 `{wiki_use_bases}=true`，额外创建 4 个 `.base` 视图：

| 路径 | 用途 |
|------|------|
| `{wiki_subdir}/views/papers.base` | papers 类页 table 视图（filter `type=paper`，按 year 排序）|
| `{wiki_subdir}/views/health-orphans.base` | 0 backlink 的 page |
| `{wiki_subdir}/views/health-stale.base` | `stale=true` 的 page |
| `{wiki_subdir}/views/health-low-confidence.base` | `confidence<0.5` 的 page |

### 命令样例

```bash
# 必需先 load obsidian-cli + obsidian-markdown skills
obsidian vault="obsidian" create path="llm-wiki/lit-wiki/SCHEMA.md" content="..." silent
obsidian vault="obsidian" create path="llm-wiki/lit-wiki/index.md" content="..." silent
# ...
```

### 完成后

返回：创建了哪些文件、`.base` 视图链接、引导用户在 Obsidian 客户端 reload 看 graph view。

---

## W8b wiki-ingest（最频繁的操作）

> Karpathy 实测：一个 source 触动 **10-15 页 wiki**——这是 ingest 的真实工作量。

### 输入

- **Zotero itemKey**（推荐——已有 metadata 和 annotations）
- 或：本地 PDF / markdown 路径（在 vault 外）
- 或：W1 刚完成的 deep-read note（最丝滑——大部分信息现成）

### 7 步流程

```
1. 读 source
   → Zotero: get_item_details + get_content(mode="complete") + get_annotations
   → 本地: Read 文件
   → 已有 W1 note: 直接读笔记内容

2. 与用户对 takeaway（防错走方向触动 10-15 页）
   → "这篇 paper 的 3 个核心 takeaway 是 ... 你认可吗？"
   → 用户 confirm 后再写

3. 扫描 wiki 现状
   → obsidian search query=<关键词> 找已有相关 page
   → obsidian read 那些 page 看现状
   → 决定：哪些 page 要 update / 哪些要新建

4. 建 paper page（必产出）
   → 路径: pages/papers/<First-Author>_<Year>_<short-slug>.md
   → frontmatter: type=paper, zotero_key=<itemKey>, sources=[zotero://...], confidence=0.85
   → 内容: TL;DR / 关键发现 / 方法 / 与本 wiki 已有 concept 的关系（用 [[wikilink]]）

5. 更新相关 entity pages（10-15 页是常态）
   → 每个 takeaway 找对应 concept/method/people page
   → 已有 → append 一段（含 [[paper page]] 反链）+ 更新 last_ingested
   → 不存在 → 新建（确认是否值得新建——警惕近重复）

6. 加双向 backlink
   → paper page → 引用的 concept/method 用 [[wikilink]]
   → concept page → 列出支持/反驳的 paper（用 [[paper page]]）

7. 追加 log.md
   → [YYYY-MM-DD] ingest | <source> | created: N pages, updated: M pages
   → 列出所有触动的 page 路径（diff-like）
```

### 用户 confirm 时机

**两次 confirm**：

| 时机 | 给用户看什么 |
|------|------------|
| Step 2 | 3 条 takeaway + 准备 ingest 的范围说明 |
| Step 7 之前 | 完整变更预览：N 个新 page + M 个 update 的 diff |

绝不批量写完才告诉用户。

### 命令样例

```bash
# 建 paper page
obsidian create path="llm-wiki/lit-wiki/pages/papers/Liu_2024_ctDNA_HRD.md" content="..." silent

# 更新已有 concept page（append 新段落）
obsidian append path="llm-wiki/lit-wiki/pages/concepts/fragmentomics.md" content="\n\n## Liu 2024 验证\n..."

# 更新 frontmatter
obsidian property:set name="last_ingested" value="2026-05-10" path="llm-wiki/lit-wiki/pages/concepts/fragmentomics.md"
obsidian property:set name="confidence" value="0.9" path="llm-wiki/lit-wiki/pages/concepts/fragmentomics.md"

# 追加 log
obsidian append path="llm-wiki/lit-wiki/log.md" content="\n[2026-05-10] ingest | Liu 2024 ctDNA HRD | created 1 paper page, updated 3 concept pages"
```

---

## W8c wiki-query（永不从记忆答）

### 7 条铁律

1. **永远先读 wiki**——不许凭训练数据印象答
2. **先读 index.md** 找相关 page 路径
3. **读相关 page 全文**再综合
4. **明确 cite page 路径**："根据 `pages/concepts/fragmentomics.md` ..."
5. **未命中明示**："wiki 里目前没有这条；要不要先 ingest 相关文献？"
6. **不臆测**：wiki 里没说的就是没说，不要补
7. **可选 follow-up**：用户深入问后，建议建新 page 沉淀这次讨论

### 流程

```
1. obsidian read path="llm-wiki/lit-wiki/index.md"
2. 解析查询 → 找相关 page 路径
3. obsidian read 每个相关 page
4. 自然语言综合 + cite page 路径
5. 列出反向链接（"X 在 [[paper-A]]、[[paper-B]] 里也提到"）
```

### 用 base 视图加速

如 `{wiki_use_bases}=true`，可借 `papers.base` / `concepts.base` 视图按 tag/year/topic 快速圈定：

```bash
obsidian base:query path="llm-wiki/lit-wiki/views/papers.base" view="by-topic" format=md
```

---

## W8d wiki-lint（健康审计）

### 三级严重度

| 级别 | 检测项 | 检测方式 |
|------|--------|---------|
| 🔴 **Critical** | orphan page（0 inbound link）| `obsidian backlinks` 或 `health-orphans.base` |
| 🔴 | 引用幻觉（claim cite 的 source 不存在）| 扫描 frontmatter sources，对每条 zotero:// 验证 |
| 🔴 | 与 source 矛盾 | 抽样对照：page claim vs source 原文（仅高 confidence page）|
| 🟡 **Warning** | `stale: true` 的页 | `health-stale.base` |
| 🟡 | `confidence < 0.5` 的页 | `health-low-confidence.base` |
| 🟡 | frontmatter 字段缺失（缺 sources / type / last_ingested）| 扫描 frontmatter |
| 🟡 | 近重复 concept page | `obsidian search` 标题相似度 + alias 表对照 |
| 🔵 **Info** | 链接密度低（<2 wikilink/页）| 扫描 page |
| 🔵 | glossary 缺词（page 出现的术语未在 glossary）| obsidian search + glossary diff |
| 🔵 | 长期未 ingest（last_ingested > 90 天）| 扫描 frontmatter |

### 输出格式

```markdown
# Lit-wiki Health Report (2026-05-10)

## 🔴 Critical (N)
- [pages/concepts/foo.md](...) — orphan page，无 backlink
  - 建议: 加 [[bar]] 链接 / archive 到 _archive/
- [pages/papers/X_2024.md](...) — frontmatter sources 中 `zotero://...ABC` 无效
  - 建议: 重新拉 Zotero metadata

## 🟡 Warning (M)
- [pages/concepts/baz.md](...) — confidence=0.4
  - 建议: 重新对照 source 提升信心或标 archived

## 🔵 Info (K)
- ...

## 总览
- 共 N pages（papers: A / concepts: B / methods: C / people: D / datasets: E）
- 平均 confidence: 0.78
- 最久未 ingest: 120 天
```

### 频次建议

每次 ingest 5-10 篇 paper 后跑一次；或 4 周一次。

---

## W8e wiki-update（手动修订单页）

### 流程

```
1. obsidian read 目标 page → 显示当前内容
2. 用户描述修订意图（"补一段关于 Tanaka 2025 的对比"）
3. agent 给 diff 预览（before / after）
4. confirm
5. 用 obsidian append / property:set 应用
6. 更新 last_ingested + content_hash
7. 追加 log.md
```

### 注意

- **如果是 source 更新（PDF 改了 / Zotero metadata 改了）触发的修订**：先标 `stale: true`，更新内容后再标回 `stale: false` + 更新 `content_hash`
- **不要直接覆盖整个 page**：用 append + property:set 增量改，保留版本演进感

---

## Frontmatter Schema（每页固定字段）

```yaml
---
title: "ctDNA fragmentomics for HRD detection"   # 必填
type: concept                                    # 必填，paper | concept | method | person | dataset
confidence: 0.85                                 # 必填，0-1
last_ingested: 2026-05-10                        # 必填，YYYY-MM-DD
sources:                                         # 必填，至少 1 条
  - zotero://select/library/items/ABC123
  - zotero://select/library/items/DEF456
content_hash: 8f3a9c                             # 必填，sources 内容的 hash 摘要（前 6-8 位）
stale: false                                     # 必填，bool
tags: [HRD, ctDNA, fragmentomics, lit-wiki]      # 必填，至少 [lit-wiki]
zotero_key: ABC123                               # 仅 type=paper 必填
aliases: ["fragmentome analysis"]                # 可选，concept 类常用，配合 disambiguation
---
```

**强约束**：
- 缺字段 → wiki-lint 标 🟡 Warning
- `sources` 不许空——claim 必须有出处
- `confidence` 永远是 agent 估的；用户可手工调

---

## 5 类页面规范

### `pages/papers/<First-Author>_<Year>_<short-slug>.md`

```markdown
---
title: ...
type: paper
zotero_key: ABC123
sources: [zotero://select/library/items/ABC123]
...
---

# {Full Title}

**Citation**: {Authors} ({Year}). *{Journal}* {Vol}({Issue}):{Pages}. DOI: {DOI}
**Zotero**: zotero://select/library/items/{key}

## TL;DR
≤2 句话

## 关键发现
- F1 ([[concept-X]] 验证；Fig 2a)
- F2 ([[method-Y]] 应用；Table 1)

## 方法
- 数据：[[dataset-Z]]
- 关键技术：[[method-Y]]

## 与 wiki 现状的关系
- 验证 [[concept-X]]
- ⚠ 与 [[paper-W]] 在低 TF 灵敏度上分歧（详见 [[concept-X]]）
- 扩展 [[method-Y]] 至 BRCA cohort

## Backlinks
（自动维护：哪些 concept/method/synthesis page 引用本页）
```

### `pages/concepts/<kebab-case-slug>.md`（最高价值）

```markdown
---
title: "ctDNA fragmentomics"
type: concept
aliases: ["cell-free DNA fragmentomics", "fragmentome analysis"]
...
---

# ctDNA Fragmentomics

## 一句话定义
基于血浆 cfDNA 片段长度分布、末端基序、覆盖度等物理特征做检测/分类的方法。

## 关键 claim（按 confidence 排序，每条带 cite）
- AUC 0.85+ 在高 TF（>5%）样本（[[Liu_2024]] AUC=0.89, [[Watkins_2022]] AUC=0.91）

> [!warning] 低 TF 子集分歧
> [[Liu_2024]] 报 78%，[[Chen_2023]] 报 45%。可能源于 cohort（复发 vs 一线）+ panel 深度差异。

## 相关
- 父概念: [[liquid-biopsy]]
- 子概念: [[end-motif-analysis]], [[fragment-length-distribution]]
- 相关方法: [[shallow-WGS]], [[targeted-panel]]
- 相关疾病: [[homologous-recombination-deficiency]]

## 来源 paper
（自动维护：所有 cite 本 concept 的 paper page）
```

### `pages/methods/<kebab-case-slug>.md`

类似 concept，但聚焦"做法"。frontmatter `type=method`。

### `pages/people/<First> <Last>.md`

```markdown
---
title: "Jane Liu"
type: person
affiliation: "Stanford"
...
---

# Jane Liu

## 研究方向
- ctDNA fragmentomics
- HRD detection

## 关键工作
- [[Liu_2024]] (Nature Cancer)
- [[Liu_2022]] (Nature Med)

## 合作者
- [[John Chen]]（共同一作）
- [[Yu Wang]]（共同通讯）
```

### `pages/datasets/<kebab-case-slug>.md`

```markdown
---
title: "TCGA-OV"
type: dataset
size: "N=587"
...
---

# TCGA-OV (Ovarian Cancer Cohort)

## 描述
The Cancer Genome Atlas Ovarian Serous Cystadenocarcinoma cohort.

## 关键信息
- N=587 (HGSOC)
- WGS / WES / RNA-seq / methylation

## 用过它的工作
- [[Liu_2024]]: 用作 validation cohort
- [[Chen_2023]]: 用作 discovery cohort
```

---

## Wiki 写作规范

### Wikilink 优先

跨页引用一律 `[[page-slug]]` 而非 markdown 链接。例：

```markdown
✅ 该方法在 [[Liu_2024_ctDNA_HRD]] 中验证 AUC=0.89
❌ 该方法在 [Liu 2024](pages/papers/Liu_2024_ctDNA_HRD.md) 中验证
```

理由：Obsidian graph view + 自动 rename + backlinks 系统都依赖 wikilink。

### Callouts 标语义

| Callout | 用途 |
|---------|------|
| `> [!quote]` | 引原文段落 |
| `> [!warning]` | 标分歧 / 与已有 wiki 矛盾 |
| `> [!example]` | 具体应用案例 |
| `> [!info]` | 元信息（dataset 规模、cohort 描述）|
| `> [!faq]` | 常见疑问 |

### Embed 数据视图

把 `.base` 视图嵌进 markdown：

```markdown
![[papers.base#by-year]]
```

适合放在 `overview.md`、`README.md`、各 concept 页结尾"来源 paper"section。

### 反 AI 味儿（继承 W6 黑名单）

- 黑名单 25 词 + 中文套话同 `review.md`
- wiki 写作风格**比综述更克制**：条目式、不抒情、不堆形容词
- 段落优先短句；callout 替代连续段落
- 出现的所有 claim 必须能 cite 回 source

---

## 反幻觉污染机制

Karpathy 的痛点：**LLM 一次 invent 的链接持续被强化**——每次 ingest 都看到那个错链，反复确认它"存在"。

### 4 重防御

1. **content_hash 检测**
   - source 文件 hash 变了 → page 自动标 `stale: true`
   - lint 报告 stale page 让用户决定 update / archive

2. **strict citation enforcement**
   - 每个 claim 必带 source（zotero:// 或 raw 路径 + 页码 / Fig）
   - cite 不可达 → lint 报 🔴 critical
   - 不许凭训练数据印象添加 claim（即使"看起来对"）

3. **两阶段 ingest**
   - Step 2 先和用户对 takeaway，再写
   - 错走方向时用户能在 takeaway 阶段拦下，不致触动 10-15 页

4. **disambiguation 规则**
   - 在 SCHEMA.md 定义概念命名约定 + alias 表
   - 新建 concept 前先 search 已有 + alias，避免近重复
   - lint 检测标题相似度 + alias 重叠

---

## 与 W1-W7 的协作

**永远 opt-in**——W8 不自动触发，由用户主导。

| 既有工作流完成时 | W8 介入方式 |
|-----------------|-----------|
| **W1 精读笔记** | 询问："要 ingest 到 lit-wiki 吗？" → 同意 → W8b（直接复用 W1 笔记内容）|
| **W2 综合笔记** | 询问：综合笔记的 themes 是否 promote 为 concept page → 选定 themes → W8b |
| **W5 外源发现入库** | Zotero 入库后询问是否同步建 wiki paper page → W8b |
| **W6 综述完成** | 询问综述识别出的 emerging concepts 是否写为 concept page |
| **W7 原创论文** | Methods/Results 引用的关键概念询问是否查 lit-wiki 已有 concept page；写完后询问是否把本研究的 finding 加入 wiki |

---

## 错误处理

| 情况 | 处理 |
|------|------|
| obsidian CLI 不可用（vault 未打开 / Obsidian 未启动）| "请先打开 Obsidian 并加载目标 vault" + 暂停 W8 操作 |
| `{wiki_subdir}` 不存在 | 提示用户先跑 W8a wiki-init |
| 写入冲突（手动改了同一文件）| 报告 conflict，停止；让用户在 Obsidian 客户端处理 |
| Zotero source 不可达（zotero:// URL 无效）| 标 page sources 那条为 [失效]，不阻塞流程；lint 报告 |
| 一次 ingest 触动 >20 页 | 警告"超出常态范围（通常 10-15）"，问用户是否继续——可能 source 太宽泛或 wiki 结构问题 |
| 用户让 wiki 改其他目录 | 拒绝："W8 范围限制在 `{wiki_subdir}`；要改 `40-文献阅读/` 等请用普通 obsidian-cli 操作" |
| obsidian skill 未 load 就调 `obsidian` CLI | 自我中止；先通过 Skill 工具 load `obsidian-cli` + `obsidian-markdown` |

---

## 不要做的事

- ❌ 不要直接用 Write/Edit 工具写 vault 路径——必须走 obsidian-cli
- ❌ 不要不读 wiki 就答 W8c 查询（凭记忆答 = 幻觉污染入口）
- ❌ 不要批量 ingest 不给 takeaway confirm（错走方向 10-15 页）
- ❌ 不要新建近重复 concept page（先 search + alias 检查）
- ❌ 不要 silently 改用户已有非 wiki 文件
- ❌ 不要修订 source（raw paper / Zotero metadata）——wiki 是 distillation，source 不可变
- ❌ 不要忽略 stale 标记——stale page 必须先评估再用
- ❌ 不要 invent zotero:// URL / DOI / 文献 metadata——cite 不到的就标"未验证"
