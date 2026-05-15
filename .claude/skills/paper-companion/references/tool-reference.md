# zotero-mcp 工具速查

zotero-mcp 暴露 22 个工具。本文件按"读 / 写 / Collection 管理"三类组织，列出每个工具的 **何时用 / 关键参数 / 典型调用 / 注意事项**。

## 调用前缀

所有工具的真实调用名为 `mcp__zotero-mcp__<short_name>`。下方各章节用 short name 简化阅读，实际工具调用时务必带前缀。

完整 22 个调用名清单：

```
mcp__zotero-mcp__get_item_details
mcp__zotero-mcp__get_item_abstract
mcp__zotero-mcp__get_content
mcp__zotero-mcp__get_annotations
mcp__zotero-mcp__get_collections
mcp__zotero-mcp__get_collection_items
mcp__zotero-mcp__get_collection_details
mcp__zotero-mcp__get_subcollections
mcp__zotero-mcp__search_library
mcp__zotero-mcp__search_collections
mcp__zotero-mcp__search_fulltext
mcp__zotero-mcp__search_annotations
mcp__zotero-mcp__fulltext_database
mcp__zotero-mcp__write_item
mcp__zotero-mcp__write_metadata
mcp__zotero-mcp__write_note
mcp__zotero-mcp__write_tag
mcp__zotero-mcp__create_collection
mcp__zotero-mcp__update_collection
mcp__zotero-mcp__delete_collection
mcp__zotero-mcp__add_items_to_collection
mcp__zotero-mcp__remove_items_from_collection
```

## 读取类（13 个）

### 元数据与全文

#### `get_item_details`
- **何时用**：拿到 itemKey 后查文献完整 metadata（标题、作者、年份、DOI、附件列表、tags、笔记列表）
- **关键参数**：`itemKey`（必需）、`mode`（minimal / preview / standard / complete）
- **典型调用**：`get_item_details(itemKey="ABC123", mode="standard")`
- **用于**：W1 起点；W4 引用生成的元数据来源

#### `get_item_abstract`
- **何时用**：只要摘要，不要其他元数据。API 比 `get_item_details` 快
- **关键参数**：`itemKey`、`format`（json / text）
- **典型调用**：`get_item_abstract(itemKey="ABC123", format="text")`
- **注意**：摘要可能为空（用户没填、或元数据没抓到）

#### `get_content`
- **何时用**：读 PDF 全文 + 笔记 + 附件文字
- **关键参数**：
  - `itemKey`（取整篇所有内容）或 `attachmentKey`（取单个附件）
  - `mode`（minimal 500 / preview 1.5K / standard 3K / **complete 无限制**）
  - `include`：`{pdf, notes, abstract, attachments, webpage}` 各项 true/false
  - `format`：json（带 metadata）/ text（纯文本）
- **典型调用**（W1 精读必用）：`get_content(itemKey="ABC123", mode="complete", include={pdf:true, notes:true, abstract:true})`
- **注意**：mode="complete" 可能返回很长内容（数十万 tokens），酌情分段处理

#### `get_annotations`
- **何时用**：读用户在 PDF 里做的高亮 / 批注 / 笔记
- **关键参数**：`itemKey`、`colors`（按颜色过滤：yellow/red/green/blue/purple/orange）、`tags`、`types`（note/highlight/annotation/ink/text/image）
- **典型调用**：`get_annotations(itemKey="ABC123", types=["highlight", "note"])`
- **用于**：W1 精读时复用用户已有标注；引用必须保留原文措辞

### Collection / 树结构

#### `get_collections`
- **何时用**：列文献库的 collection
- **关键参数**：`recursive`（true 取整棵树，含嵌套）、`parentCollection`、`mode`
- **典型调用**：`get_collections(recursive=true)`
- **注意**：递归模式返回完整树，分页参数会被忽略

#### `get_collection_items`
- **何时用**：列某个 collection 里的文献条目
- **关键参数**：`collectionKey`（必需）、`limit`、`offset`
- **典型调用**：`get_collection_items(collectionKey="XUTGBNVQ", limit=50)`
- **用于**：W2 综合时圈定范围

#### `get_collection_details`
- **何时用**：取单个 collection 的详细信息（名字、parent、子集数量等）
- **关键参数**：`collectionKey`

#### `get_subcollections`
- **何时用**：取某 collection 的子 collection（不含文献）
- **关键参数**：`collectionKey`、`recursive`

### 搜索

#### `search_library`
- **何时用**：综合搜索文献，支持标题 / 全文 / 类型 / 年份 / 排序
- **关键参数**：
  - `q`（通用查询）、`title`（标题搜索）、`fulltext`（PDF/note 全文）
  - `fulltextMode`：attachment / note / both
  - `itemType`：journalArticle / book / preprint / attachment 等
  - `yearRange`：如 `"2020-2024"`
  - `sort`：relevance / date / title / year
  - `relevanceScoring`：true 启用相关度评分
  - `mode`：minimal 30 / preview 100 / standard 自适应 / complete 500+
- **典型调用**：`search_library(q="HRD ctDNA", yearRange="2022-2025", sort="relevance", relevanceScoring=true)`
- **用于**：W3、W4 起点

#### `search_collections`
- **何时用**：按名字找 collection
- **关键参数**：`q`、`limit`

#### `search_fulltext`
- **何时用**：在 PDF 全文里找关键字（不是搜文献，是搜"哪些 PDF 提到过 X"）
- **关键参数**：`q`、`itemKeys`（限定范围）、`caseSensitive`、`contextLength`、`maxResults`
- **典型调用**：`search_fulltext(q="signature 3", contextLength=200)`
- **注意**：返回片段含上下文，可直接展示给用户

#### `search_annotations`
- **何时用**：在用户标注里搜（"我有没有标过关于 X 的笔记"）
- **关键参数**：`q`、`colors`、`tags`、`itemKeys`、`minRelevance`

#### `fulltext_database`
- **何时用**：访问全文缓存数据库（比 `search_fulltext` 快）
- **关键参数**：`action`（list / search / get / stats）、`query`、`itemKeys`

## 写入类（4 个）

⚠ **所有写入前必须先把内容/参数给用户 confirm，得到肯定再调用。**

#### `write_note`
- **何时用**：W1 写精读笔记 / W2 写综合笔记 / 任意场景给文献加笔记
- **关键参数**：
  - `action`：create / update / append
  - `content`：Markdown 字符串（自动转 HTML）
  - `parentKey`：附在某文献下（child note）；省略则建 standalone note（适合 W2 综合笔记）
  - `noteKey`：update / append 时必需
  - `tags`：数组
- **典型调用**：
  ```
  # W1 child note
  write_note(action="create", parentKey="ABC123", content="# TL;DR\n...", tags=["auto:deep-read"])
  # W2 standalone
  write_note(action="create", content="# HRD 综合笔记\n...", tags=["auto:synthesis", "auto:topic:hrd"])
  ```

#### `write_item`
- **何时用**：创建新文献条目（仅元数据，不含 PDF），或把已有 standalone PDF 重新挂到新条目下
- **关键参数**：
  - `action`：create / reparent
  - `itemType`：journalArticle / book / preprint / thesis / conferencePaper / report / webpage / bookSection 等
  - `fields`：`{title, abstractNote, date, DOI, url, volume, issue, pages, publicationTitle, ...}`
  - `creators`：`[{creatorType: "author", firstName: "...", lastName: "..."}]`
  - `tags`、`attachmentKeys`、`parentKey`
- **典型调用**：
  ```
  write_item(
    action="create",
    itemType="journalArticle",
    fields={title:"...", date:"2024", DOI:"10.x/y", publicationTitle:"Nature"},
    creators=[{creatorType:"author", firstName:"Jane", lastName:"Doe"}],
    tags=["auto:registered"]
  )
  ```
- **注意**：不能上传本地 PDF，PDF 必须先在 Zotero 客户端导入成 standalone attachment

#### `write_metadata`
- **何时用**：修改已有条目的元数据（修标题、补 DOI、改作者列表）
- **关键参数**：`itemKey`（必需）、`fields`、`creators`（替换全部作者）
- **注意**：只对 regular item 有效，不能改 note 或 attachment

#### `write_tag`
- **何时用**：给条目加 / 删 tag
- **注意**：详细 schema 用到时再加载（ToolSearch select）

## Collection 管理（5 个）

#### `create_collection`
- **何时用**：用户明确说要建新 collection
- **关键参数**：`name`、`parentCollection`（嵌套时填）

#### `update_collection`
- **何时用**：重命名或移动 collection
- **关键参数**：`collectionKey`、`name`、`parentCollection`（""=移到顶层）

#### `delete_collection`  ⚠ 危险
- **何时用**：用户明确请求且 **double-confirm** 后
- **关键参数**：`collectionKey`、`deleteItems`（true 把文献也送回收站；默认 false 只删 collection）
- **默认禁止使用**——除非用户主动且明确请求

#### `add_items_to_collection`
- **何时用**：把文献加入某 collection
- **关键参数**：`collectionKey`、`itemKeys`（数组）

#### `remove_items_from_collection`  ⚠ 半危险
- **何时用**：从某 collection 移出文献（不删除条目）
- **关键参数**：`collectionKey`、`itemKeys`
- **注意**：用户说"删除这篇文献"时**不要**用这个——这只是从 collection 移除，不是真删。要明确告诉用户区别

## 全工具能力总结

- ✅ 读：metadata / abstract / 全文 / 高亮 / collection / 搜索
- ✅ 写：笔记 / 元数据 / tag / 创建条目（无 PDF）
- ✅ 组织：collection 增删改、文献加/移
- ❌ 不能：上传本地 PDF / 删除文献条目 / 合并重复条目 / 修改用户已有 annotation
