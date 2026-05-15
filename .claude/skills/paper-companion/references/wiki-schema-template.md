# Lit-Wiki SCHEMA

> 这是 W8a wiki-init 生成的 `{wiki_subdir}/SCHEMA.md` 的内容模板。复制到 vault 后，agent 后续操作时按本文件规则维护 wiki。

---

# Lit-Wiki Schema & 维护守则

本文件定义 lit-wiki 的结构、命名、frontmatter、引用规则。**LLM agent 在维护本 wiki 时必须遵守**。如规则与某次操作冲突，停下来与用户对齐再改 SCHEMA。

## 1. 目录结构

```
lit-wiki/
├── SCHEMA.md          ← 本文件
├── README.md          ← 给人读的说明
├── index.md           ← 主目录（按 type 分组）
├── log.md             ← 操作日志
├── overview.md        ← 顶层综述（live）
├── glossary.md        ← 术语表
├── pages/
│   ├── papers/        ← 1 paper = 1 page
│   ├── concepts/      ← 跨 paper 的概念
│   ├── methods/       ← 方法/技术
│   ├── people/        ← 研究者
│   └── datasets/      ← 数据集/cohort
└── views/             ← .base 数据视图
```

## 2. 5 类页面 + 命名约定

| Type | 路径 | 命名约定 | 例 |
|------|------|---------|---|
| paper | `pages/papers/` | `<First-Author>_<Year>_<short-slug>.md` | `Liu_2024_ctDNA_HRD.md` |
| concept | `pages/concepts/` | `kebab-case-english.md` | `fragmentomics.md` |
| method | `pages/methods/` | `kebab-case.md` | `shallow-WGS.md` |
| person | `pages/people/` | `<First> <Last>.md`（保留空格）| `Jane Liu.md` |
| dataset | `pages/datasets/` | `kebab-case.md` | `TCGA-OV.md` |

新建 page 前**必须 search 已有 + alias**，避免近重复。

## 3. Frontmatter（必填字段）

```yaml
---
title: "..."                    # 必填
type: paper|concept|method|person|dataset
confidence: 0.0-1.0             # agent 自评
last_ingested: YYYY-MM-DD
sources:                        # ≥1 条
  - zotero://select/library/items/<key>
content_hash: <6-8 hex>         # sources 内容 hash 摘要
stale: true|false
tags: [..., lit-wiki]           # 必含 lit-wiki

# type=paper 额外:
zotero_key: <key>

# type=concept 推荐:
aliases: [...]
---
```

缺字段 → lint 标 🟡 Warning。

## 4. 引用规则（critical）

### 4.1 跨页引用一律 wikilink

```markdown
✅ [[Liu_2024_ctDNA_HRD]] 报 AUC=0.89
❌ [Liu 2024](pages/papers/Liu_2024_ctDNA_HRD.md) 报 AUC=0.89
```

### 4.2 Claim 必带 source

```markdown
✅ AUC=0.89 (Liu 2024, Fig 2a)
❌ AUC 大约 0.89
```

### 4.3 不许凭训练数据印象添加 claim

agent 不能基于"我记得文献里说过"添加任何内容——必须有 source page 支持。

### 4.4 zotero:// URL 是 source 主要形式

Obsidian 支持 `zotero://select/library/items/<key>` 协议，点击直接打开 Zotero 对应条目。

## 5. Disambiguation（防近重复）

### 5.1 新建 concept 前

1. `obsidian search` 标题关键词
2. 检查已有 concept page 的 `aliases:`
3. 命中相似 → 直接 update 现有 page，不要新建

### 5.2 已知 alias 映射

> 用户 / agent 在维护过程中持续补充：

- `fragmentomics` ⊃ `cfDNA fragmentomics`, `fragmentome analysis`, `片段化分析`
- `HRD` ⊃ `homologous recombination deficiency`, `同源重组缺陷`
- `ctDNA` ⊃ `circulating tumor DNA`, `循环肿瘤 DNA`
- ...

## 6. Wikilink Backlinks 约定

每个 page 末尾应有 "Backlinks" 段落（Obsidian 自动维护，但模板要留位）：

```markdown
## Backlinks
（自动：哪些 page 链到本页）
```

## 7. Callouts 语义

| Callout | 用途 |
|---------|------|
| `> [!quote]` | 引原文段落 |
| `> [!warning]` | 标分歧 / 与已有 wiki 矛盾 |
| `> [!example]` | 具体应用案例 |
| `> [!info]` | 元信息 |
| `> [!faq]` | 常见疑问 |

## 8. Lint 触发条件

| 级别 | 条件 |
|------|------|
| 🔴 Critical | orphan page / 引用 source 不可达 / 与 source 矛盾 |
| 🟡 Warning | stale=true / confidence<0.5 / frontmatter 字段缺失 / 近重复 |
| 🔵 Info | 链接密度 <2/页 / glossary 缺词 / 长期未 ingest（>90 天）|

## 9. 操作日志格式

`log.md` 每条 1 行：

```
[2026-05-10] init
[2026-05-10] ingest | Liu 2024 ctDNA HRD | created 1 paper, updated 3 concepts
[2026-05-11] update | concepts/fragmentomics.md | refined low-TF section
[2026-05-12] lint | 2 critical, 5 warning, 12 info
```

## 10. 反 AI 味儿（继承 paper-companion W6）

- 25 个英文黑名单词 + 6 类中文套话——见 `paper-companion/references/review.md`
- wiki 风格**比综述更克制**：条目式、不抒情、不堆形容词

## 11. 范围硬边界

W8 操作仅限 `lit-wiki/` 目录内。**不得修改** vault 内其他目录。如需操作其他目录，stop + 询问用户是否需要走非 W8 流程。

## 12. 非 W8 写入

如果用户在 Obsidian 客户端**手工**修改了 wiki 文件，agent 下次 ingest 时应：

1. 检测到 last_modified 比 last_ingested 新
2. 询问用户："这页有手工修改，是要保留还是被新 ingest 覆盖？"
3. 默认保留——不要 silently 覆盖

---

**本 SCHEMA 由 paper-companion W8a wiki-init 生成于 {YYYY-MM-DD}。修改前请理解每条规则的意图——SCHEMA 是 wiki 维护质量的根。**
