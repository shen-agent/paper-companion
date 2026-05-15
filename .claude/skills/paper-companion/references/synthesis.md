# 跨文献主题综合 (W2)

读 N 篇文献，输出主题**综合笔记**（共识 / 分歧 / 方法学比较 / 开放问题），回写为 Zotero standalone note。

> **术语澄清**：本工作流的输出叫**「综合笔记」**——是个人研究性质的跨文献整合笔记，**不等于发表级综述文章**（manuscript-style review）。后者由 InsightLab 后续的专门 skill 处理。
>
> 用户口语中往往说"综述"——agent 应识别为 W2 触发词，但产出仍按"综合笔记"模板（短小、研究导向、可在数十分钟内完成）。若用户实际想要发表级综述（结构、长度、文献广度都不同），应建议切换到对应 skill 或让用户澄清。

> **Agent 参数引用**：本文中 `{tag_prefix}` / `{note_language}` / `{content_mode}` 来自 `agents/paper-companion-agent/agent.yaml`。默认值：`tag_prefix=auto:`、`note_language=zh`、`content_mode=complete`。

## 工作流（5 步 + confirm）

```
1. 确认范围
   → 询问用户：综合笔记基于什么？
     - collection（"基于 HRD collection"）→ get_collection_items(collectionKey)
     - tag（"所有标了 auto:deep-read 的文献"）→ search_library(q="...", 用 tag 过滤)
     - 显式列表（"这 5 个 itemKey"）→ 直接进入 step 2
     - 主题查询（"过去 3 年关于 ctDNA fragmentomics 的"）→ search_library(q="...", yearRange="2022-2025")
   → 命中数 < 3 时提醒"样本太少，综合笔记可能不可靠"
   → 命中数 > 15 时让用户筛选或分主题
   → 给用户列表（标题 + 作者 + 年份），让其确认

2. 逐篇读全文
   → 对每个 itemKey:
     - get_item_details(itemKey, mode="standard")     # 元数据 + 是否已有 deep-read note
     - 优先：若已有 child note 标了 "auto:deep-read"，先 get_content 这条 note 复用
     - 否则：get_content(itemKey, mode="complete") 读全文
   → 边读边在内部记 4 类信号：
     - 共识点（多数文献一致的结论）
     - 分歧点（不同文献相反/不同的主张）
     - 方法学差异（数据规模、技术、对照设计）
     - 开放问题（未解决的争议、留给后续工作的）

3. 按模板生成综合笔记草稿（见下方"模板"）

4. **给用户 confirm**
   → 展示完整 markdown 草稿
   → 询问："综合笔记是否需要调整？哪段内容偏了？是否还要纳入其他文献？"
   → 等待用户确认（"可以"、"写入 Zotero"）

5. 回写为 standalone note
   → write_note(
       action="create",
       # 注意：不传 parentKey，做成 standalone note
       content=<markdown>,
       tags=["{tag_prefix}synthesis", "{tag_prefix}topic:<主题slug>"]
     )
   → 告诉用户：综合笔记以 standalone note 形式写入，可在 Zotero 主目录看到
```

## 综合笔记模板

```markdown
# {主题}综合笔记（生成于 {YYYY-MM-DD}）

**Scope**: 基于 N 篇文献。来源：{collection X / tag Y / 显式列表 / 查询 "..."}

**Papers**:
1. {Author1} et al. ({Year1}) — {Journal1}
2. {Author2} et al. ({Year2}) — {Journal2}
...

## 共识
- **C1**：…（来自 {paperA}, {paperB}, {paperC}）
- **C2**：…（来自 {paperX}, {paperY}）

## 分歧与争议
- **D1**：{paperA} 主张 X；{paperB} 主张 Y
  - **冲突点**：…（数据 / 方法 / 解释）
  - **可能原因**：…（cohort 差异 / 技术敏感度 / 时间窗口）
- **D2**：…

## 方法学比较

| 文献 | 数据规模 | 关键技术 | 对照 | 关键结论 |
|------|---------|----------|------|----------|
| {paperA} | N=… | … | … | … |
| {paperB} | N=… | … | … | … |
...

## 综合判断与开放问题

- **当前主流**：…
- **未解决**：…
- **下一步可能方向**：…
- **建议精读 / 复现**：{paperX}（理由：…）
```

## 写作规则

1. **共识不是简单"都说了 X"**
   - 必须是**经多个独立来源验证**的结论
   - 单篇支持的不是共识，写到分歧或开放问题
   - 每条共识至少标注 ≥2 篇 paper

2. **分歧要写清"冲突点"**
   - 不要只说"A 说 X，B 说 Y"
   - 要写：是数据不同？方法不同？解释角度不同？技术敏感度不同？
   - 这部分是综合笔记真正的价值所在

3. **方法学比较表必须有**
   - 即使只有 3-5 篇，也列出来
   - 让读者对比"数据规模 / 技术 / 对照"差异 → 自己判断哪些结论更可信

4. **开放问题要具体**
   - 不要写"未来还有很多工作要做"——废话
   - 写：哪个具体假设没人验证？哪个 cohort 类型没人覆盖？哪个对照缺失？

5. **Tag 必加两个**
   - `{tag_prefix}synthesis` （类型）
   - `{tag_prefix}topic:<主题slug>` （主题）
   - 例：`auto:topic:hrd-detection`

6. **综合笔记写为 standalone note**（无 `parentKey`）
   - 因为综合笔记跨多篇，不属于任何单篇
   - Standalone note 在 Zotero 主目录直接列出，方便检索

## 范围确认对话样例

> **用户**："给 HRD 这个 collection 写个综述"
>
> **agent**：
> （识别用户意图为 W2 跨文献综合，输出"综合笔记"——不是发表级综述文章。如用户实际想要后者，会另作说明。）
> 我先列一下 HRD collection 里的文献：
> 1. Liu 2024 — Nature Cancer（关于 ctDNA HRD signature）
> 2. Chen 2023 — Genome Medicine（panel 设计）
> 3. Kumar 2022 — JCO（临床验证）
> ...（共 8 篇）
>
> 综合笔记将基于这 8 篇。是否要：
> - 全部纳入？
> - 排除某几篇（如年份过早）？
> - 增加 collection 外的若干 itemKey？
>
> 另外：是否需要先读这些 paper 的精读笔记（如果已有）？还是直接读全文？

## 错误处理

| 情况 | 处理 |
|------|------|
| 命中文献 = 1 | 不是综合笔记场景，建议走 W1 精读单篇 |
| 命中文献 = 2 | 警告"样本量太小，综合笔记意义有限"，问用户是否继续 |
| 命中文献 > 20 | 警告并建议先按子主题分桶；或让用户 cherry-pick |
| 部分文献无 PDF/全文 | 用 abstract 兜底；在综合笔记顶部标注"基于 N 篇全文 + M 篇仅摘要" |
| 多数文献已有 deep-read note | 优先复用 note，比重读全文快得多 |

## 不要做的事

- ❌ 不要把每篇文献的 TL;DR 拼起来当综合笔记——那是清单不是综合
- ❌ 不要回避分歧、把不同结论拼成"互补观点"——这是抹平
- ❌ 不要跨主题硬凑（"HRD + 单细胞 + AI 大模型" 三个不相干的不能合一篇综合笔记）
- ❌ 不要在没有 confirm 范围的情况下直接动手读 N 篇——浪费 token

## 完整范例

基于虚构 5 篇文献做的 ctDNA HRD 检测主题综合笔记：

```markdown
# ctDNA HRD 检测综合笔记（生成于 2026-05-09）

**Scope**: 基于 5 篇文献。来源：HRD collection（XUTGBNVQ）

**Papers**:
1. Liu et al. (2024) — Nature Cancer
2. Chen et al. (2023) — Genome Medicine
3. Watkins et al. (2022) — Nature Medicine
4. Kumar et al. (2022) — JCO
5. Tanaka et al. (2025) — Cell Reports Medicine

## 共识
- **C1 ctDNA fragmentomics 可量化 HRD**：高 TF（>5%）样本 AUC>0.85（Liu, Watkins, Tanaka）
- **C2 HRD+ 患者 PARP 响应更好**：PFS 显著延长（Liu, Kumar）
- **C3 panel 需结合片段长度 + 末端基序**：单一信号不足（Chen, Watkins, Tanaka）

## 分歧与争议

- **D1 低 TF（<3%）样本灵敏度**：Liu 报 78%，Chen 报 45%
  - **冲突点**：cohort 阶段（复发 vs 一线）+ panel 深度（5× vs 2×）+ 末端基序数量（128 vs 64）
  - **可能原因**：复发患者 ctDNA 释放不稳定但片段化更显著

- **D2 BRCA-like 表型**：Tanaka 主张 ctDNA 能识别非 BRCA 突变 HRD；Liu / Chen 仅在 BRCA 突变 cohort 验证

## 方法学比较

| 文献 | 数据规模 | 关键技术 | 对照 | 关键结论 |
|------|---------|----------|------|----------|
| Liu 2024 | N=120, 复发 | fragmentomics + end motif | myChoice CDx | AUC=0.89 |
| Chen 2023 | N=200, 一线 | 50-gene panel | myChoice CDx | AUC=0.84 |
| Watkins 2022 | N=350, 混合 | fragmentomics | 组织 HRD score | AUC=0.91 |
| Kumar 2022 | N=85, PARP 治疗 | ctDNA HRD | RECIST 响应 | HR=0.38 |
| Tanaka 2025 | N=180, 无 BRCA | 综合特征 | scarHRD | AUC=0.82 |

## 综合判断与开放问题

- **当前主流**：高 TF cohort 中 ctDNA fragmentomics 是 HRD 量化可靠替代方案
- **未解决**：低 TF（<3%）灵敏度优化；非 BRCA HRD 表型识别；多中心一致性
- **下一步可能方向**：低 TF 增敏算法；循环 RNA / 5hmC 多组学融合；亚洲人群验证
- **建议精读 / 复现**：Watkins 2022（fragmentomics 框架根源，AUC 最高）；用我们 panel N=80 重做 Liu 2024 低 TF 子集
```

**注意范例的几个特点**：
- 每条共识都标注 ≥2 篇支持（C1: 3 篇，C2: 2 篇，C3: 3 篇）
- 分歧不只是"A 说 X B 说 Y"——还分析冲突点和可能原因
- 方法学比较表是综合笔记真正价值所在：让读者自己判断哪些结论更可信
- "下一步方向"具体到算法 / 多组学 / 人群，不写"未来还有很多工作"
- "建议精读"给具体理由（"AUC 最高"、"用我们 panel 重做"）

