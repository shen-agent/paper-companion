# 数据驱动的原创论文生成 (W7)

把用户已有的数据 + 分析结果组织成符合 IMRaD 格式的可投稿 manuscript 草稿。区别于 W6（综述别人的研究），W7 写**用户自己的**原创研究。

> **Agent 参数引用**：本文中 `{citation_style}` / `{note_language}` / `{target_journal}` / `{study_type}` 来自 `agents/paper-companion-agent/agent.yaml`。默认值：`citation_style=informal`、`note_language=zh`、`target_journal=未指定`、`study_type=未指定`。

> **不在作用域**：实验设计 / 数据采集 / 统计分析（这些应该已经做完了；用户带来 results 给 agent）。Agent 不替用户做统计、不替用户决定研究设计。

> **核心承诺**：W7 不发明数据、不编造结果、不替用户拍板研究问题。Agent 是**写作伙伴**而非研究者——把用户已有的客观结果用规范、严谨、无 AI 味儿的语言组织成 manuscript。

## 工作流（8 步）

### Step 1 — 投稿目标 + 研究类型定义

询问用户：
- **论文类型**：original article / brief communication / case report / methods paper / perspective
- **目标期刊**：决定字数上限、参考文献上限、abstract 格式（结构化 vs 非结构化）、citation style
  - 不确定时让用户列 2-3 个候选；agent 帮查体例
- **研究设计**：随机对照 / 队列 / 病例对照 / 横断面 / 诊断准确性 / 预测模型 / 动物实验 / 病例报告 / 其他
  - 这一步决定 reporting guideline（见下方"Reporting Guidelines"）
- **预期 takeaway**（一句话）：本研究最重要的发现是什么——这是后续所有章节对齐的锚点

输出：scope 摘要 + 选定的 reporting guideline + 字数预算 → confirm。

### Step 2 — 数据 / 分析现状盘点

用户提供：
- **Raw data 路径**（CSV / Excel / RDS / fastq 等）
- **分析脚本**（R / Python / Stata / SAS 路径）
- **已生成的 tables**（baseline characteristics、主要结果表、亚组分析等）
- **已生成的 figures**（KM 曲线、ROC、Forest plot 等）路径
- **关键 findings 列表**（用户口述：3-5 条 bullet，每条带具体数字）
- **Ethics**：IRB 批件号、知情同意状态

Agent 整理出 **Results-ready 输入清单**：

| 资产 | 内容 | 用于 Methods | 用于 Results | 待补 |
|------|------|------------|------------|------|
| Table 1 | baseline | ✓ 描述 | ✓ 表 1 | n/a |
| KM curve | OS by group | n/a | ✓ Fig 2 | 是否做了 log-rank? |
| ... | ... | ... | ... | ... |

→ confirm。

### Step 3 — 风格校准

同 W6：问"1-3 篇你过往的论文 / 目标期刊近期同类型论文"做 voice 样本。

**比 W6 多一步**：让用户指定 **目标期刊近期 1-2 篇同类型论文**，提取期刊体例特征：
- abstract 是结构化（Background/Methods/Results/Conclusions）还是单段
- Methods 是单一节还是分小节（Participants / Procedures / Statistical Analysis）
- Results 是否带小标题
- Discussion 长度比例（通常 25-35% 全文）
- Reference 渲染（Vancouver / APA / numbered / Author-Year）
- 缩写表是否需要

### Step 4 — Outline（按写作顺序）

**写作顺序（critical，违反顺序会增加 AI 味儿）**：

```
Methods → Results → Discussion → Conclusion → Intro → Abstract → Title
```

为什么这个顺序：
- Methods + Results 最 data-grounded，AI 套话最难混进——先写
- Discussion 必须知道完整 results 才能展开
- Intro 是给读者"接上"已经写好的故事——最后铺垫
- Abstract 浓缩需全局视角
- Title 最后定（往往最难推敲）

每节 outline 用 bullets，标注：关键 claim / 引用 / Fig / Table / 字数预算。

→ confirm outline。

### Step 5 — 草稿生成（按写作顺序，分章节渐进 confirm）

每节生成后**立即扫黑名单 + 验证引用**，再交给用户。绝不批量生成全文一次性给。

#### 5a. Methods（按 reporting guideline 逐项填）

**框架**（按选定 guideline 调整子标题）：

```markdown
## Methods

### Study Design
（设计类型 + 时间 + 地点 + 注册号如有）

### Participants / Sample
（纳入排除标准、招募方式、样本量计算依据、N=...）

### Data Collection / Procedures
（具体到试剂版本、设备型号、SOP 链接、操作时间窗）

### Outcomes
（主要终点 + 次要终点的精确定义；如何测量）

### Statistical Analysis
（每个分析的统计方法 + 软件版本 + 关键参数 + 假设检验 + 多重检验校正 + 缺失值处理）

### Ethics
（IRB 批件号 + 知情同意 + 数据保密；动物实验加 3R）
```

**Methods 反 AI 规则**：

| ❌ AI 套话 | ✅ 具体 |
|-----------|--------|
| "we conducted comprehensive analysis" | "we ran logistic regression with elastic net regularization (alpha=0.5, λ via 10-fold CV; glmnet 4.1-7, R 4.3.2)" |
| "appropriate statistical methods were used" | 直接列：t-test / Wilcoxon / DeLong test / multivariable Cox |
| "samples were processed using standard protocols" | 给协议引用："per Liu et al. (2020) protocol with the modification that..." |
| "data were carefully cleaned" | "missing values (n=12, 4.3%) were imputed via MICE; outliers (>3 SD) excluded after sensitivity check" |
| "various confounders were considered" | 直接列调整变量：age, sex, BMI, smoking status, prior treatment |

**关键要求**：
- 详细到**他人能复现**——这是 Methods 最高标准
- 软件 / 试剂必须带版本号
- 统计方法必须有显著性水平 + 单 / 双侧
- 涉及 RNA / cell line / antibody → 来源 + cat#

→ confirm Methods。

#### 5b. Results（用户数字直接落地）

**框架**：

```markdown
## Results

### Participant Characteristics（描述 cohort，引 Table 1）

### Primary Outcome（主要终点结果，引主要 Fig/Table）

### Secondary Outcomes（次要终点）

### Subgroup Analyses（亚组）

### Sensitivity Analyses（敏感性分析；如有）
```

**Results 反 AI 规则**（最严格）：

- ❌ "we observed significant differences"
- ✅ "AUC=0.89 (95% CI 0.84–0.93) vs 0.78 (0.72–0.84) for control (p<0.001, DeLong test; Fig 2a)"

**强制项**（每个 claim）：
- N（样本量）
- Effect size（OR / HR / RR / Δ / d / r）
- 95% CI 或 SE
- p value（精确到 3 位小数；p<0.001 不写 p=0.000）
- 引用 Fig X 或 Table X
- 过去时

**严禁**：
- 解读（"this suggests / implies"）→ 留给 Discussion
- 重复 Table 数字（用文字总结趋势 + 引 Table）
- 引入新数据（Results 里出现的 N 必须 Methods 已说过）
- 使用 "significantly" 而不给 p value

→ confirm Results。

#### 5c. Discussion

**框架（4 段式经典结构）**：

```markdown
## Discussion

第 1 段：主要发现概括（不重复 Results，1-2 句新表述）+ 在更大 context 中的位置

第 2 段：与已有文献对比（≥3 篇关键文献；显式说出一致 / 不一致点；不抹平分歧）

第 3 段：机制假说 / 临床意义 / 方法学意义（具体到数字推论，不写"重要意义"）

第 4 段：局限性（≥3 项，每项 specific）+ 未来方向（每项有具体路径）

最后短段：一句话结论
```

**Discussion 反 AI 规则**：

| ❌ AI 套话 | ✅ 具体 |
|-----------|--------|
| "our findings have important implications" | "AUC 提升 Δ=0.11 → 若应用于 Liu (2024) 的 N=2000 cohort，预计可多识别 220 例阳性" |
| "future research should explore" | "下一步应在多中心前瞻队列（N≥500）验证；尤其需覆盖 BRCA-like 而非仅 BRCA mutated" |
| "this study has several limitations" | 直接列编号：(i) 单中心 cohort；(ii) 18 个月随访不足以评估 OS；(iii) 缺乏 PARP-naive 对照 |
| "the results are consistent with previous studies" | "AUC=0.89 与 Watkins (2022) 报告的 0.91 在 95% CI 内重叠；但与 Chen (2023) 的 0.84 显著不同（详见下方）" |

→ confirm Discussion。

#### 5d. Conclusion（≤150 词）

一句话核心发现 + 一句话临床/方法学意义 + 一句话最关键开放问题。**不要**重述结果细节。

#### 5e. Introduction

**框架**：

```markdown
## Introduction

第 1 段：临床/科研问题的重要性（具体数字：发病率、死亡率、社会成本）+ 当前主流方法

第 2 段：当前方法的局限（具体）+ 已有改进尝试（关键文献综述，3-5 篇核心工作；不要堆砌）

第 3 段：本研究的 niche / gap（一句话精确刺中）+ 本研究做了什么（与本文 Methods 一致的高度概括）+ 本研究的贡献（避免"novel/groundbreaking"，写具体新颖点）
```

**Intro 反 AI 规则**：
- 第 1 段开头不要"近年来" / "随着 X 的发展" / "In recent years"——用具体数据刺破
- 文献综述不要 "Several studies have shown..."——直接 "Liu (2024) 报 X；Chen (2023) 报 Y"
- gap 陈述要精确：不是"现有方法不够好"，是"现有方法在 TF<3% 子集灵敏度仅 45%（Chen 2023），导致..."

→ confirm Intro。

#### 5f. Abstract

按目标期刊格式：
- **结构化**（多数医学期刊）：Background / Methods / Results / Conclusions，各部分严格字数（如 NEJM 50/100/100/50 = 250 词）
- **非结构化**（Nature 子刊、PLOS 等）：单段 ≤ 250 词，叙述流

强制：与正文数字 100% 一致；abstract 出现的所有 claim 在正文有支持。

→ confirm Abstract。

#### 5g. Title

≤ 15 词；体现研究问题或主要发现。

3 个候选 → 让用户选。**避免**：
- "A study of X"
- "Investigation into Y"
- "Novel / groundbreaking / first-of-its-kind"
- 问句标题（除非期刊文化允许）

→ confirm Title。

### Step 6 — Tables / Figures Captions

**Caption 必须 self-explanatory**：读者不读正文也能理解。

模板：
```
Table/Fig X. [描述性标题，one line]. [方法摘要 1-2 句]. [N + 关键统计标注]. [缩写定义].
```

例：
> **Table 1.** Baseline characteristics of study participants (N=120). Continuous variables shown as median (IQR); categorical as n (%). Comparisons by Wilcoxon rank-sum (continuous) or Fisher's exact (categorical). HRD, homologous recombination deficiency; TF, tumor fraction.

**规则**：
- 不重复正文数字（用文字趋势总结）
- 1 个 per ~1000 词正文（4 节正文 → 4-6 个 tables/figures 是合理上限）
- 缩写在 caption 首次出现处定义
- 引用工具版本（如 ggplot2 4.x、ComplexHeatmap）

### Step 7 — 8 维质量审查

每节生成后跑 + 全文完成跑总检：

| 维度 | 标准 |
|------|------|
| **Reporting guideline** | 选定 guideline（CONSORT/STROBE/...）的所有 items 已覆盖（提供 checklist 报告）|
| **引用完整性** | 每条 cite 经 CrossRef 验证（fabrication=0, orphan cites=0）|
| **数据完整性** | 每个 Results claim 都有 N + effect size + CI + p value + Fig/Table 引用 |
| **反 AI 味儿** | 黑名单 0 命中（继承 W6 黑名单 + Methods/Results/Discussion 各自规则）|
| **章节比例** | Discussion 通常 25-35%；Methods 20-30%；Results 25-35%；Intro 10-15%；偏离 >10% 时告警 |
| **Tables/Figures** | 数量 ≤ 1/1000 词；caption self-explanatory；不与正文重复 |
| **Tense** | Methods/Results 过去时；established facts 现在时；Discussion 过去 + 现在混 |
| **Abbreviations** | 全部首次定义；如期刊要求加缩写表 |

任何维度 FAIL → 修订 → 重检。3 次 FAIL 同一维度 → 暴露给用户决定。

### Step 8 — 输出

写入位置：
- **本地**（默认）：`manuscript_<topic-slug>_<YYYYMMDD>.md` 或 `.tex`
- **可选**：分章节文件（`01_intro.md`、`02_methods.md` ...）便于协作 / 版本控制
- **可选**：cover letter draft（按目标期刊体例）
- **可选**：CONSORT/STROBE checklist 填好的对照表（投稿时 supplementary 用）

返回：路径 + 字数（按节细分）+ 引用条数 + 8 维质检报告 + 选定 reporting guideline 的覆盖度。

## Reporting Guidelines 速查（Methods 必依赖）

| 研究类型 | Guideline | items 数 | 关键内容 | 必备图 |
|---------|-----------|---------|---------|-------|
| **随机对照试验** | [CONSORT 2010](http://www.consort-statement.org/) | 25 | 随机化方法、盲法、序列产生、分配隐藏 | CONSORT flow diagram |
| **观察性（cohort/case-control/cross-sectional）** | [STROBE](https://www.strobe-statement.org/) | 22 | 偏倚控制、混杂调整、亚组分析 | 选择流程图 |
| **诊断准确性** | [STARD 2015](https://www.equator-network.org/reporting-guidelines/stard/) | 30 | 真阳/假阳定义、阈值选择、indeterminate 处理 | STARD flow diagram |
| **预测模型** | [TRIPOD](https://www.tripod-statement.org/) | 22 | derivation/validation 划分、calibration、AUC + 95% CI | calibration plot |
| **动物实验** | [ARRIVE 2.0](https://arriveguidelines.org/) | 21 | 物种/品系、笼养、3R | n/a |
| **病例报告** | [CARE](https://www.care-statement.org/) | 13 | 时间线、知情同意 | 时间线图 |
| **质量改进** | [SQUIRE 2.0](http://www.squire-statement.org/) | 18 | 干预描述、context、可推广性 | n/a |
| **临床试验方案** | [SPIRIT 2013](https://www.spirit-statement.org/) | 33 | 入组流程、interim 分析、stop rules | SPIRIT flow |
| **经济评价** | [CHEERS 2022](https://www.equator-network.org/reporting-guidelines/cheers/) | 28 | 时间窗、贴现率、敏感性分析 | n/a |

完整的 checklist 见各 guideline 官网。Agent 在 Step 1 选定后，Step 5a Methods 写作时按对应 checklist 逐项核对。

## Tense 速查

| 章节 | 主要时态 | 例外 |
|------|---------|------|
| Abstract | 过去时（结果）+ 现在时（结论）| — |
| Introduction | 现在时（established facts）+ 过去时（指代具体 study）| 本研究的 statement of purpose 现在时 |
| Methods | **过去时**（"we measured", "samples were collected"）| — |
| Results | **过去时**（"the AUC was 0.89", "patients showed"）| 描述图表用现在时（"Fig 2a shows"）|
| Discussion | 过去时（指本研究 finding）+ 现在时（解读 / 推论 / 普遍 claim）| — |
| Conclusion | 现在时为主 | — |

## Tables vs Figures 决策

| 数据特征 | 选 Table | 选 Figure |
|---------|---------|----------|
| 精确数字（如 baseline characteristics）| ✓ | |
| 多变量横向对比 | ✓ | |
| 趋势 / 时间序列 | | ✓ |
| 分布 / 形态 | | ✓ |
| 生存曲线 / KM | | ✓ |
| ROC / calibration | | ✓ |
| 多组比较的 effect size + CI | | ✓ Forest plot |
| 流程 / 分组 | | ✓ flowchart |

## 引用验证（同 W6 硬规则）

每条 cite 强制 `mcp__paper-search-mcp__get_crossref_paper_by_doi(doi=...)` 验证。LLM 实测 31% 引用错误率不能接受——不许凭"训练数据印象"引用。

DOI 不可达的标 `[DOI 未验证]`，让用户决定是否人工补。

## 三层任务委派（同 W6）

| 层 | W7 任务 | agent 行为 |
|----|---------|-----------|
| **Tier 1 自动** | 章节模板填空、reference 渲染、字数统计、Caption 模板 | 直接做 |
| **Tier 2 协作** | Methods/Results 草稿（基于用户数字）、Discussion 推论、CONSORT/STROBE 自查 | 给草稿 → user confirm |
| **Tier 3 苏格拉底** | 研究 niche 定位、机制假说、亮点提炼、亚组分析意义 | 主动反问，不替用户拍板 |

## 三大失败模式 + 缓解（W7 专属）

| 失败 | 表现 | 缓解 |
|------|------|------|
| **数字伪造** | LLM 生成"看起来合理"但不在用户数据里的 N、p、AUC | 强制规则：Results 中所有数字必须能在用户提供的 tables/scripts 中找到出处；不许"compute 一下大致" |
| **Methods 含糊化** | 把用户的具体协议改写成 "standard procedures were followed" | 反 AI 规则强制具体到版本号 / cat# / 参数；Methods reproducibility 是金标准 |
| **过度声明** | Discussion 推论超出数据支持范围（如单中心 N=120 推 "applicable to all populations"）| 每个 claim 必须在 Results 找到对应支持；Discussion 第 4 段强制 ≥3 项具体局限 |

## 完整范例（Methods + Results 节选）

下方是虚构 ctDNA HRD 研究的 **Methods 一节 + Results 一段**，展示反 AI 味儿 + 严谨：

```markdown
## Methods

### Study Design
This was a single-center, retrospective cohort study conducted at [Institution] between January 2020 and June 2024. The protocol was approved by the [IRB Committee, approval #2019-1234] and registered at ClinicalTrials.gov (NCT04XXXXXX). All participants provided written informed consent.

### Participants
We enrolled 120 patients with histologically confirmed BRCA1/2-mutated high-grade serous ovarian cancer who had received first-line platinum-based chemotherapy. Inclusion required (i) ≥18 years, (ii) measurable disease per RECIST 1.1, and (iii) plasma sample collection within 30 days of imaging. We excluded patients with concurrent malignancy or prior PARP inhibitor exposure.

### Sample Processing
Plasma was separated within 2 hours of phlebotomy by double centrifugation (1,600×g for 10 min, then 16,000×g for 10 min) and stored at −80 °C. Cell-free DNA was extracted using the QIAamp Circulating Nucleic Acid Kit (QIAGEN, cat# 55114) per manufacturer protocol. Library preparation followed the KAPA HyperPrep kit (Roche, KK8504) with custom xGen probes (IDT) covering 50 HRD-related genes.

### Statistical Analysis
The primary endpoint was area under the receiver operating characteristic curve (AUC) for HRD detection, with tissue myChoice CDx HRD score ≥42 as ground truth. Sample size of 120 provided 90% power to detect AUC ≥0.85 vs null of 0.70 at two-sided α=0.05. AUCs were compared using DeLong's test. We calibrated logistic regression with elastic net regularization (alpha=0.5, λ via 10-fold cross-validation) using glmnet 4.1-7 in R 4.3.2. Subgroup analysis stratified by tumor fraction (TF) used pre-specified thresholds (<3%, 3–5%, >5%). Missing covariate values (n=8, 6.7%) were imputed via multivariate imputation by chained equations (mice 3.16). Two-sided p<0.05 was considered statistically significant; we did not adjust for multiple comparisons given the exploratory nature of subgroup analyses.

## Results

### Cohort Characteristics
Of 145 patients screened, 120 met eligibility (Fig 1). Median age was 58 years (IQR 51–65); 84/120 (70%) carried BRCA1 mutations. Baseline characteristics are detailed in Table 1.

### Primary Outcome
The composite ctDNA HRD signature yielded AUC=0.89 (95% CI 0.84–0.93) for distinguishing tissue HRD-positive from HRD-negative samples (Fig 2a). This outperformed the single fragment-length feature (AUC=0.78, 0.72–0.84; ΔAUC=0.11, p<0.001 by DeLong test) and matched the gold-standard tissue panel within margin (sensitivity 87% vs 91%, specificity 84% vs 89%).
```

**注意上文**：
- 0 个黑名单词
- 每个数字（N、试剂版本、统计参数）都具体
- 软件 + 试剂带版本号 / cat#
- IRB 号 + ClinicalTrials.gov ID
- Results 中每个 claim 带 N + effect size + CI + p value + Fig 引用
- 全过去时（Methods 和 Results）
- "we did not adjust for multiple comparisons given the exploratory nature"——主动声明而非含糊
- 段落开头变化：claim、数字、对比，不重复

## 错误处理

| 情况 | 处理 |
|------|------|
| 用户给的 finding 没有具体数字 | 暂停 Results 写作；让用户补具体值（"AUC 大概 0.85" 不够，要精确值 + CI） |
| Results 中需要的统计 Methods 没说 | 倒推让用户在 Methods 补；不许在 Results 引入 Methods 没声明的方法 |
| 选定 guideline 的某 item 用户没数据 | 在 Methods 显式标注 "not collected" / "not reported"；不要伪造 |
| 投稿期刊未指定 | 用领域中性默认（IMRaD + 结构化 abstract + Vancouver citation）+ 可后期切换 |
| 字数严重超 / 不足目标期刊 | 第一时间告知；先给方向（哪段最该精简 / 扩展），让用户决定 |
| 用户的统计方法选择争议 | 不替用户拍板；建议用户咨询统计师；agent 写出他选定的方法 |

## 不要做的事

- ❌ 不要发明数据、p value、CI——所有数字必须在用户提供的资产中可追溯
- ❌ 不要替用户做研究设计判断（"应该用 Cox 还是 logistic？"）
- ❌ 不要把 Methods 含糊化成 "standard procedures"——具体到版本号 / cat# / 参数
- ❌ 不要在 Results 解读（"this suggests"）——留给 Discussion
- ❌ 不要在 Discussion 推论超出数据支持范围
- ❌ 不要凭"训练数据印象"引用文献——必须 CrossRef 验证
- ❌ 不要一次性写完 7 节给用户——按 Methods → Results → ... 顺序分章节 confirm
- ❌ 不要伪造 ethics 信息（IRB 号 / ClinicalTrials.gov ID）
- ❌ 不要把"草稿"包装成"已完稿"——它是 manuscript draft，需用户深度修订 + 共同作者 review
