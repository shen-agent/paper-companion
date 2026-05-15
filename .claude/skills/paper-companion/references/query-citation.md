# 查询与引用 (W3 / W4)

W3：自然语言搜索 → 自然语言总结（不返 raw JSON）
W4：根据 itemKey 或搜索结果，按指定 citation_style 渲染引用

> **Agent 参数引用**：本文中 `{citation_style}` / `{note_language}` 来自 `agents/paper-companion-agent/agent.yaml`。默认值：`citation_style=informal`、`note_language=zh`。用户没显式指定时按 agent 默认。

## W3 自然语言查询

### 工作流

```
1. 解析意图
   → "搜 X" / "我库里有没有 Y" / "找 Z 相关文献" → search_library
   → "在 PDF 里找过 X 的句子" → search_fulltext
   → "我有标过 X 的高亮吗" → search_annotations
   → "X 这个 collection 里有什么" → get_collection_items

2. 调用搜索
   search_library(
     q="...",
     yearRange="2020-2025",          # 视情况
     sort="relevance",
     relevanceScoring=true,
     mode="preview"                   # 默认 100，不要过度抓取
   )

3. 根据命中数选择输出策略
   → 0 命中：解释为什么可能没找到 + 建议放宽（去掉 yearRange / 改关键词 / 去英文搜中文领域）
   → 1-5 命中：每篇详细一行（标题 + 作者+年份 + 期刊 + 一句话价值）
   → 6-20 命中：每篇一行（标题 + 作者+年份），按相关度排序，提示用户挑感兴趣的
   → >20 命中：先分桶（按 collection / 年份 / 类型），让用户选子集

4. 输出格式（自然语言，不要 raw JSON）
```

### 输出风格

**1-5 命中（详细）**：

> 找到 3 篇相关文献：
>
> 1. **Liu et al. (2024)** — *Nature Cancer*（HRD collection）
>    研究 ctDNA HRD signature 在 BRCA 突变型卵巢癌中的检测灵敏度（N=120）。
>    DOI: 10.xxx/yyy
>
> 2. **Chen et al. (2023)** — *Genome Medicine*
>    设计了一个 50-gene panel 用于 HRD 量化。验证集 AUC=0.89。
>    DOI: 10.xxx/zzz
>
> 3. ...

**6-20 命中（紧凑）**：

> 命中 12 篇，按相关度：
>
> 1. Liu 2024, Nat Cancer — ctDNA HRD signature
> 2. Chen 2023, Genome Med — 50-gene HRD panel
> 3. Kumar 2022, JCO — 临床验证 PARP 响应
> ...
>
> 想看哪几篇的细节？或要做什么操作（精读 / 综合笔记 / 引用）？

**>20 命中（分桶）**：

> 命中 47 篇。按 collection 分布：
> - HRD（28）
> - 早筛（12）
> - 我的文章（4）
> - 其他（3）
>
> 按年份：2024(15) / 2023(18) / 2022(8) / 更早(6)
>
> 要先收窄到哪个子集？还是按主题给你列子分类？

## W4 引用生成

### citation_style 4 种格式

参数 `citation_style` 来自 agent.yaml，4 种取值，对应不同输出形式。

#### `informal`（默认）

> Liu 等 (2024) 在 Nature Cancer 上报告了 …

或英文：

> Liu et al. (2024, *Nature Cancer*) showed that …

特点：作者+年份内嵌在叙述中，适合写综述、读书笔记的引文。

#### `vancouver`

> [1] Liu J, Chen X, Wang Y, et al. ctDNA HRD signature detection in ovarian cancer. *Nat Cancer*. 2024;5(3):234-245. doi:10.xxx/yyy

特点：数字编号 + 完整作者列表（>6 时用 et al.）+ 缩写期刊名 + 卷期页 + DOI。Lancet / NEJM / Cell 等期刊用。

#### `apa`

> Liu, J., Chen, X., Wang, Y., Smith, A., & Doe, J. (2024). ctDNA HRD signature detection in ovarian cancer. *Nature Cancer*, *5*(3), 234-245. https://doi.org/10.xxx/yyy

特点：所有作者倒置 (Last, F.M.) + 年份括号 + 完整期刊名（不缩写）+ DOI URL。心理学 / 教育 / 部分 Nature 子刊用。

#### `inline`

> (Liu et al., 2024)

特点：括号内 author + year 极简，适合在正文中夹引用。

### 工作流

```
1. 拿到 itemKey（用户给 / 或先 search_library 找到）

2. get_item_details(itemKey, mode="standard")
   → 拿到 title / creators / date / publicationTitle / volume / issue / pages / DOI

3. 按 citation_style 渲染
   → 注意 creators 是数组，每个元素 {creatorType, firstName, lastName} 或 {name}
   → 处理 et al. 阈值（vancouver: >6, apa: >20）
   → 期刊名缩写（vancouver）—— 没有简写时保持原名

4. 输出
   → 单篇：直接给字符串
   → 多篇：编号列表（vancouver 顺序）或字母顺序（apa）
```

### 错误与边界

| 情况 | 处理 |
|------|------|
| 缺 DOI | 渲染时省略 DOI 部分，提示用户"该条目无 DOI" |
| 缺卷期页 | 是 preprint / 在线发表？显式标注 "[Online ahead of print]" 或 "[Preprint]" |
| 作者只有 organization name | `{name: "..."}` 而非 first/last，按机构作者格式渲染 |
| 中文作者 | 尊重用户给定形式；中文期刊倾向 "姓 名" 全称不缩写 |

## 输出示例

**用户**："给我 Liu 2024 那篇用 vancouver 引用"

**agent 流程**：
```
search_library(q="Liu HRD 2024", yearRange="2024", relevanceScoring=true)
  → itemKey = ABC123
get_item_details(itemKey="ABC123", mode="standard")
  → 渲染
```

**输出**：

> Vancouver 格式：
>
> [1] Liu J, Chen X, Wang Y, et al. ctDNA HRD signature detection in ovarian cancer. *Nat Cancer*. 2024;5(3):234-245. doi:10.xxx/yyy
>
> 需要其他格式（informal / apa / inline）吗？

## 不要做的事

- ❌ 不要直接把 search 返回的 JSON 贴出来——必须自然语言总结
- ❌ 不要在引用里编造缺失的元数据（年份 / 卷期 / 作者全名）——缺什么标什么
- ❌ 不要忽略 et al. 阈值差异（vancouver 6 / apa 20 / informal 通常 ≥3 就 et al.）
- ❌ 命中过多时不要直接列 50 行——先分桶
