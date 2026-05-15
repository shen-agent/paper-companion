---
name: paper-companion
description: >
  科研文献阅读分析助手。8 类工作流：精读 / 综合笔记 / 库内查询 / 引用生成 /
  外源发现（arXiv/PubMed/Scholar）/ 文献综述 / 原创论文（IMRaD）/ 文献 wiki（Obsidian lit-wiki）。
  后端：zotero-mcp + paper-search-mcp。
  触发：精读 / 读这篇 / 综合 / 综述 / write a review / IMRaD / 写论文 / 我库里有没有 /
  生成引用 / arXiv / PubMed / 下载 PDF / lit-wiki / ingest 到 wiki / wiki 健康 /
  paper / citation / bibliography / Zotero。
  只要用户提到文献、paper、参考文献、Zotero 或要操作其文献资源即触发——Claude 倾向 undertrigger，请主动启用。
---

# 文献阅读助手 (paper-companion)

科研文献阅读分析助手。聚焦：把全文读进来、生成结构化精读笔记、跨文献综合、自然语言查询、按格式生成引用。

**当前后端**：zotero-mcp（接入 Zotero 文献库）。后续可能扩展 PubMed / arXiv / 本地 PDF 等。

## 前置条件

- **必需**：zotero-mcp 已连接（工具前缀 `mcp__zotero-mcp__*`）。验证方法：`mcp__zotero-mcp__get_collections(mode="minimal")` 能返回 collection 列表即可
- **可选（W5 必需）**：paper-search-mcp 已连接（工具前缀 `mcp__paper-search-mcp__*`），覆盖 arXiv / PubMed / Semantic Scholar / Google Scholar 等 22 个学术源
- **不支持**：上传本地 PDF（用户需手工拖入 Zotero）、删除文献条目、合并重复条目
- **写入前 confirm**：所有 `write_*` 工具调用前先把草稿/参数给用户过目——这是 zotero-mcp 工具描述里就明确要求的（"Confirm with user before writing"）

## 工作流路由

### 决策树（先判断再选 reference）

- 用户给 1 篇 → 想要笔记 → **W1**
- 用户给多篇 → 想要每篇各自的笔记 → **W1 × N**（不是 W2）
- 用户给多篇 → 想要一份跨文献综合笔记（私人自用 / 3-10 篇）→ **W2**
- 用户给多篇 → 想要可发表的 manuscript-style 综述（20-50 篇 / 反 AI 味儿 / 引用经 CrossRef 验证）→ **W6**
- 用户带**自己的数据 + 分析结果** → 想要 IMRaD 原创论文草稿（reporting guideline 驱动）→ **W7**
- 用户想把笔记 / 论文沉淀为 **跨会话累积的知识图谱**（Obsidian wiki / wikilink / health audit）→ **W8**
- 用户在 **Zotero 库内** 找文献 → **W3**
- 用户只要引用文本 → **W4**
- 用户在 **外部** 找文献 / 想从 arXiv / PubMed / Scholar 等搜 → **W5**（衔接 W1/W2/W4/W6/W7/W8）
- 零碎操作（建/改 collection、加 tag、改元数据）→ 直接读 `references/tool-reference.md` 选工具

**W3 vs W5 区分**：W3 = 已在 Zotero；W5 = 还没存的（外网）。模糊时先 W3 兜，未命中再问"要去外网搜吗？"

**W2 vs W6 区分**：
- **W2 综合笔记** = 私人研究笔记，3-10 篇，自用，标 `auto:synthesis`
- **W6 文献综述** = manuscript-style，20-50 篇，可发表草稿，引用经 CrossRef 验证，反 AI 味儿过滤，标 `auto:review`
- 用户说"综述这几篇"模糊时，按篇数判断（≤10 走 W2，≥15 走 W6），中间地带让用户选

**W6 vs W7 区分**：
- **W6** = 写**别人**的研究综述（输入：N 篇文献；输出：综述文章）
- **W7** = 写**自己**的原创论文（输入：用户的数据 + 分析结果；输出：IMRaD manuscript 草稿）
- 触发关键词：W6="综述 / review / 文献综述"；W7="写论文 / 投稿草稿 / IMRaD / Methods Results / 我的数据"

**W8 与 W1-W7 的关系**：
- W8 = **持久化层**（Obsidian lit-wiki），把 W1/W2/W6/W7 的产出沉淀为带 wikilink 的 entity pages
- W8 不替代 Zotero（Zotero 是 raw source 层），也不替代 W1-W7（W1-W7 是工作流层）
- 触发关键词："ingest 到 wiki / lit-wiki / wiki 里有没有 X / lint wiki / 检查 wiki 健康 / wiki 怎么说 X"
- W8 严格限定在 vault 内 `llm-wiki/lit-wiki/` 子目录，不修改其他用户笔记

### 路由表

| 用户语义 | 工作流 | Reference |
|---------|--------|-----------|
| "精读这篇"、"做一份深度笔记"、"读完写到 Zotero" | **W1 单篇深度精读** | `references/reading-notes.md` |
| "综述这几篇"、"对比 3-10 篇做研究笔记" | **W2 跨文献综合（私人笔记）** | `references/synthesis.md` |
| "写一篇综述"、"manuscript review"、"narrative review"、"综述章节"、"综述论文草稿" | **W6 文献综述（可发表）** | `references/review.md` |
| "用我的数据写一篇论文"、"IMRaD 草稿"、"投稿 Methods/Results"、"原创研究 manuscript" | **W7 原创论文生成** | `references/manuscript.md` |
| "我库里有没有 X"、"在 Zotero 里搜" | **W3 库内查询** | `references/query-citation.md` |
| "生成 X 篇的引用"、"整理参考文献列表" | **W4 引用生成** | `references/query-citation.md` |
| "去 arXiv / PubMed 搜"、"找最新预印本"、"下载这篇 PDF"、"DOI 转 metadata" | **W5 外源发现与入库** | `references/discovery.md` |
| "初始化 lit-wiki"、"ingest 到 wiki"、"wiki 里有没有 X"、"检查 wiki 健康"、"修订 wiki 某页" | **W8 文献 Wiki 维护** | `references/wiki.md` |

## 工具速查

- **zotero-mcp**（22 工具，前缀 `mcp__zotero-mcp__*`）：完整速查见 `references/tool-reference.md`
- **paper-search-mcp**（57 工具，前缀 `mcp__paper-search-mcp__*`）：源选择、工具列表、入库规范见 `references/discovery.md`

**只加载当前工作流所需的 reference 文件**，不要预加载全部——这是 progressive disclosure 模式的核心。

## 关键约束（细节在各 reference 中重复说明）

- **写入前必须 confirm**：`write_note` / `write_item` / `write_metadata` / `write_tag` / `create_collection` / `add_items_to_collection` 调用前展示草稿，得到用户确认再执行
- **默认禁用** `delete_collection` / `remove_items_from_collection`：除非用户明确请求并 double-confirm
- **回写笔记用 Markdown**（zotero-mcp 自动转 HTML），不要手写 HTML 标签
- **PDF 全文用 `mode="complete"`**：精读笔记不能只看摘要
