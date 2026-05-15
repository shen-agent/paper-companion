# 单篇深度精读 (W1)

读完一篇文献的全文 + 已有标注，按固定模板生成结构化笔记，回写为 Zotero child note。

> **Agent 参数引用**：本文中 `{tag_prefix}` / `{note_language}` / `{content_mode}` 来自 `agents/paper-companion-agent/agent.yaml`。默认值：`tag_prefix=auto:`、`note_language=zh`、`content_mode=complete`。下方所有形如 `{name}` 的占位符按用户的 agent 配置实际值替换。

## 工作流（5 步 + confirm）

```
1. 定位文献
   → 用户给 itemKey：直接进入 step 2
   → 用户只描述："读一下 Liu 2024 那篇 HRD 文章"
     → search_library(q="Liu HRD", yearRange="2024", sort="relevance", relevanceScoring=true)
     → 命中多篇时让用户挑

2. 拉取上下文
   → get_item_details(itemKey, mode="standard")        # 元数据 + 附件 + 已有 note 列表
   → get_content(itemKey, mode="complete")             # PDF 全文（必须 complete）
   → get_annotations(itemKey)                          # 用户已有的高亮（可能空）

3. 按模板生成笔记草稿（见下方"模板"）

4. **给用户 confirm**
   → 展示完整 markdown 草稿
   → 询问："这版精读笔记是否需要调整？哪部分要补充或删减？"
   → 等待用户回复（"可以"、"OK"、"写入"）

5. 回写 Zotero
   → write_note(
       action="create",
       parentKey=<itemKey>,
       content=<markdown>,
       tags=["{tag_prefix}deep-read"]
     )
   → 确认成功后告诉用户："已写入 Zotero，可在客户端 {Title} 条目下看到这条笔记"
```

## 笔记模板（必须严格遵循）

```markdown
# {Title}

**Citation**: {Authors} ({Year}). *{Journal}* {Vol}({Issue}):{Pages}. DOI: {DOI}

## TL;DR
≤2 句话讲清"做了什么、最重要的发现、为什么重要"。

## 背景与问题
为什么做。要回答的具体问题。1-3 段。

## 方法
- **数据**：N=…（cohort 来源、样本类型）
- **关键技术**：…（数据处理流程、模型/算法、平台）
- **对照**：…（baseline、comparator group）

## 关键发现
- F1：… (Fig X / Table Y, p<…)
- F2：…
- F3：…

## 与已有研究的关系
- **验证**：…（哪些已有结论被本研究确认）
- **反驳/冲突 ⚠**：…（与哪些先前文献结论相反——必须显式标记，不抹平）
- **扩展**：…（在哪个方向上推进）

## 局限与开放问题
- 样本偏倚 / 未做的对照 / 可复现性 / 后续工作

## 我的备注
- 是否值得引用？引用场景？
- 后续动作（精读哪些它引用的工作？做哪个对照实验？）
- 相关 idea
```

## 写作规则（6 条硬约束）

1. **TL;DR 控制 ≤2 句**
   - 写不出来意味着还没读懂——回去重读
   - 不要堆形容词（"开创性的"、"重要的"），讲具体做了什么

2. **关键 claim 必须 cite 回原文位置**
   - 形如 `(Fig 2a)`、`(Table 1)`、`(§Results, p.4)`、`(p<0.001)`
   - 让未来的自己/读者能 5 秒内核实

3. **第三人称视角**
   - 不写"这篇文章很有意思"、"作者很厉害"——这是读书日记
   - 写"该研究通过 X 方法证明 Y"——这是知识条目

4. **冲突显式标记 `⚠`，不抹平**
   - 与你已读过的、已有 note 的、已知主流观点矛盾的，必须标 `⚠` + 简述冲突点
   - 不要为了叙述顺滑而调和——分歧本身是知识

5. **Tag 自动加 `{tag_prefix}deep-read`**
   - `tag_prefix` 来自 agent 参数（默认 `"auto:"`）
   - 完整 tag 例：`auto:deep-read`
   - 让用户能一眼区分"我手工写的笔记" vs "agent 生成的笔记"

6. **写入前先草稿 + confirm**
   - 永远不要跳过 confirm 直接 `write_note`
   - 草稿在聊天里完整展示 markdown，让用户看到最终落到 Zotero 的内容

## 错误处理

| 情况 | 处理 |
|------|------|
| `get_content` 返回空（无附件 / 无 PDF） | 改 `get_item_abstract` + 询问用户是否仍要写笔记（仅基于摘要的版本，质量打折）|
| PDF 文本含大量 OCR 噪声 | 在草稿里说明可信度受限，让用户确认是否继续 |
| `get_annotations` 返回空 | 正常，跳过该步——多数文献没人标注 |
| 摘要也为空 | 让用户给 DOI 或 PDF 路径，agent 不能凭空写 |
| `write_note` 报错 `Invalid parentKey` | 检查 itemKey 是否是 regular item（不能是 attachment 或另一个 note 的 key）|

## 不要做的事

- ❌ 不要复制粘贴摘要充当 TL;DR
- ❌ 不要写"本文亮点 / 创新点 / 重要意义"等营销腔
- ❌ 不要在笔记里嵌图（zotero-mcp 不支持图片附件，HTML 也不能引用本地图）
- ❌ 不要省略 confirm 直接 write_note
- ❌ 不要用 `parentKey` 指向 collection key 或 note key——只能是 regular item key

## 完整范例

下面是对虚构文献 *Liu et al. (2024) ctDNA HRD signature in ovarian cancer* (Nature Cancer) 做精读笔记的实际产物，展示模板各字段的填法和"好笔记"长啥样。

```markdown
# ctDNA HRD signature in ovarian cancer

**Citation**: Liu J, Chen X, Wang Y, et al. (2024). *Nature Cancer* 5(3):234-245. DOI: 10.1038/s43018-024-00789-0

## TL;DR
该研究在 120 例 BRCA 突变型卵巢癌中验证基于 ctDNA 片段化模式的 HRD 检测，AUC=0.89，将 HRD 量化窗口从组织活检拓展到外周血。

## 背景与问题
组织 HRD 检测是 PARP 抑制剂用药决策金标准，但对手术不可行或复发患者难以实施。本研究探索 ctDNA 是否能等效替代。

## 方法
- **数据**：120 例 BRCA1/2 突变型卵巢癌，配对外周血 + 组织样本，shallow WGS（~5×）
- **关键技术**：ctDNA 片段长度分布 + 末端基序谱（end motif），logistic regression 分类
- **对照**：以组织 myChoice CDx HRD score ≥42 为金标准

## 关键发现
- F1：训练集 AUC=0.92，验证集 AUC=0.89（Fig 2a, n=40）
- F2：低肿瘤分数（TF<3%）样本灵敏度仍达 78%（Fig 3b）
- F3：HRD+ 患者 PFS 延长 5.4 个月（HR=0.42, p<0.001, Fig 4）

## 与已有研究的关系
- **验证**：与 Watkins 2022 (Nat Med) 的 ctDNA fragmentomics 框架一致
- **反驳/冲突 ⚠**：Chen 2023 (Genome Med) 报告类似 panel 在低 TF 样本灵敏度仅 45%——本研究高 33pp。可能因 cohort 差异（Liu 用复发患者 TF 中位 8% vs Chen 一线 2%）
- **扩展**：首次将 ctDNA HRD 量化与 PARP 响应直接关联

## 局限与开放问题
- 单中心 cohort，未做多中心验证
- 缺乏非 BRCA 突变型 HRD 表型样本（如 BRCA-like）
- 长期随访仅 18 个月

## 我的备注
- 值得引用：用作"ctDNA 可替代组织 HRD"的参考
- 后续动作：精读 Watkins 2022 验证 fragmentomics 一致性
- 待复现：低 TF 子集灵敏度（用我们 panel 数据 N=80 重做）
```

**注意范例的几个特点**：
- TL;DR 1 句话讲清楚做了什么 + AUC 数字 + 价值
- 每个发现都有图编号 + p 值 / 数字
- 与已有研究的反驳点显式标 ⚠ 并解释冲突原因（不是简单"A 说 X B 说 Y"）
- "我的备注"含具体下一步动作（精读哪篇、复现哪个实验），不写"很有意义"

