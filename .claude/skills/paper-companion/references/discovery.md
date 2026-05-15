# 文献发现与入库 (W5)

通过 paper-search-mcp 在 22 个学术源中搜文献，可选下载 PDF / 读全文 / 入库 Zotero。这是 paper-companion 与外部知识源的桥梁——区别于 W3（只搜 Zotero 本地库）。

> **Agent 参数引用**：本文中 `{tag_prefix}` / `{note_language}` 来自 `agents/paper-companion-agent/agent.yaml`。默认值：`tag_prefix=auto:`、`note_language=zh`。

## 工作流（4-6 步）

```
1. 选源
   → 按主题 / 已知信息选最合适的源（见下方"源选择速查"）
   → 不确定时用 search_papers（通用分发）或 search_semantic（Semantic Scholar，全学科覆盖最广）

2. 搜索
   → search_<source>(query=..., limit=N) 拿候选列表
   → 命中 >20 时让用户筛选

3. 用户挑选 / 缩窄
   → 自然语言列出前 5-10 篇（标题 + 作者 + 年份 + 源）
   → 询问用户：要(a) 看摘要 / (b) 读全文 / (c) 下载 PDF / (d) 入库 Zotero / (e) 上述组合？

4. 读取或下载（按用户选择）
   - **A 看摘要**：search 返回里通常已含 abstract，直接呈现
   - **B 读全文**：`read_<source>_paper(...)` 直接拿正文
   - **C 下载 PDF**：`download_<source>(target_path, ...)` 存到本地
   - **D 多源容错**：`download_with_fallback(...)` 多个源依次尝试

5. 入库 Zotero（可选）
   → write_item(action="create", itemType="journalArticle"|"preprint", fields={...}, creators=[...], tags=["{tag_prefix}from:<source>"])
   → ⚠ PDF 不能通过 zotero-mcp 自动上传——告知用户需手工拖入 Zotero 客户端，或导入 BibTeX
   → 入库后返回 itemKey，可衔接 W1/W2/W4

6. 衔接后续工作流（可选）
   → 单篇精读 → 走 W1（itemKey 已有）
   → 多篇综合 → 走 W2
   → 只要引用 → 走 W4
```

## 源选择速查

| 学科 / 场景 | 首选 | 备选 |
|------------|------|------|
| 生物医学（已发表）| `search_pubmed` / `search_pmc` | `search_europepmc` / `search_crossref` |
| 生物医学（预印本）| `search_biorxiv` / `search_medrxiv` | `search_openalex` |
| 计算机 / AI / ML | `search_arxiv` | `search_dblp` / `search_semantic` |
| 数学 / 物理 | `search_arxiv` | `search_openalex` / `search_semantic` |
| 加密 / 安全 | `search_iacr` | `search_arxiv` |
| 经济 / 社科 | `search_ssrn` | `search_doaj` / `search_semantic` |
| 法语论文 | `search_hal` | — |
| 已知 DOI 补 metadata | `get_crossref_paper_by_doi` | `search_semantic` |
| 找开放获取 PDF | `search_unpaywall` | `search_doaj` / `search_openaire` / `search_openalex` |
| 全学科通用 | `search_semantic` / `search_google_scholar` | `search_papers` / `search_openalex` |
| 数据集 / 软件 | `search_zenodo` | — |
| 学位论文 / 灰色文献 | `search_base` / `search_core` | `search_citeseerx` |

## 22 个源 + 57 个工具

每个源至少有 `search_*`，多数有配套 `read_*_paper` 和 `download_*`。完整工具调用名前缀：`mcp__paper-search-mcp__`。

**所有源**（按字母）：
arxiv、base、biorxiv、citeseerx、core（仅 search）、crossref、dblp、doaj、europepmc（仅 search）、google_scholar（仅 search）、hal、iacr、medrxiv、openaire、openalex、pmc（仅 search）、pubmed、scihub（仅 download）、semantic、ssrn、unpaywall（仅 search）、zenodo

**特殊工具**：
- `search_papers` — 通用分发器，自动选源
- `download_with_fallback` — 多源容错下载
- `get_crossref_paper_by_doi` — DOI → metadata（W4 引用补全的最佳路径）

## ⚠ Sci-Hub 红线

`download_scihub` 存在但**默认禁止调用**——Sci-Hub 法律地位有争议，许多机构政策禁用。

仅在用户**明确请求**并 confirm 知晓风险时使用，调用时在回复里加免责说明。不要把 Sci-Hub 作为 `download_with_fallback` 的备选源。

## 范围确认对话样例

> **用户**："找几篇 ctDNA fragmentomics 的最新预印本"
>
> **agent**：这是预印本场景，我用 bioRxiv + medRxiv 搜近 1 年的。
>
> 找到 5 篇：
>
> 1. Liu 2025 (bioRxiv) — End motif diversity in plasma cfDNA for early cancer detection
> 2. Chen 2024 (medRxiv) — Fragmentomics-based pan-cancer screening (N=2000)
> 3. Watanabe 2025 (bioRxiv) — Tumor-derived cfDNA fragment size signatures
> ...
>
> 想做：
> - (a) 看每篇摘要
> - (b) 读 1-2 篇全文
> - (c) 全部下载 PDF 到 `D:/papers/`
> - (d) 全部入库 Zotero（PDF 你后续手工拖入）
> - (e) 选 1-2 篇做精读笔记（衔接 W1）

## 错误处理

| 情况 | 处理 |
|------|------|
| 单源命中 0 | 切到 `search_papers` 或 `search_semantic`（覆盖最广）；或扩宽关键词 |
| 多源结果重复 | 按 DOI 去重；CrossRef 作为权威 metadata 兜底 |
| `read_*_paper` 返回不完整（部分预印本只有 abstract）| 改 `download_*` 拿 PDF，再交给用户读 |
| 下载失败 | `download_with_fallback` 多源尝试；最终失败建议人工或 Open Access 链接 |
| 用户要 Sci-Hub | 一次性 confirm + 免责说明，不默认调用 |
| 入库后用户问"PDF 怎么没了" | 解释 zotero-mcp 不支持 PDF 上传；建议拖入或 BibTeX 导入 |

## 入库时的元数据规范

调用 `write_item` 时，按文献来源补 fields：

| 字段 | 来源 |
|------|------|
| `title` | search 返回 |
| `creators` | 用 `{creatorType:"author", firstName, lastName}` 数组 |
| `date` | 发表/预印日期 |
| `DOI` | 优先 |
| `url` | 源链接 |
| `publicationTitle` | 期刊名（已发表）|
| `archive` | "arXiv" / "bioRxiv" / "medRxiv"（预印本时设这个）|
| `extra` | 可放 arXiv ID / PMID / 其他外部 ID |
| `tags` | 至少加 `{tag_prefix}from:<source>`（标记来源便于追溯）|

`itemType` 选择：
- 已正式发表 → `journalArticle`
- 预印本 → `preprint`
- 会议论文 → `conferencePaper`
- 学位论文 → `thesis`
- 数据集 / 软件 → `dataset` 或 `computerProgram`

## 不要做的事

- ❌ 不要默认调用 `download_scihub`
- ❌ 不要在用户没 confirm 前批量下载（浪费带宽+磁盘）
- ❌ 不要伪造 DOI / metadata（缺什么字段就标"未知"，让用户决定是否补）
- ❌ 入库时忘记 `tag_prefix` 前缀
- ❌ 假装能上传 PDF——明确告知限制，给替代方案
