# 基于循环游离DNA的多癌种早期检测：从分子信号到临床转化

**作者**：[用户姓名]
**日期**：2026-05-11
**Generated with**: paper-companion W6 (Claude); 引用经 CrossRef 验证（未做 voice 校准，使用领域中性学术风格）

---

## Abstract

**Background.** 全球每年新发癌症超过 1900 万例，超过 70% 的癌症相关死亡发生在中低收入国家（WHO, 2022）。多数实体肿瘤在 I 期确诊时 5 年生存率超过 90%，而 IV 期降至 10%-30%；现有筛查手段（如低剂量 CT、结肠镜、乳腺 X 线）仅覆盖少数癌种且依从性有限。循环游离 DNA（cfDNA）作为一种微创液体活检标志物，为同时筛查多种癌症提供了可能。

**Objective.** 本综述系统梳理 2016-2026 年间 33 篇代表性工作，聚焦三个问题：cfDNA 携带哪些可利用的早筛分子信号？不同检测技术路线的性能如何对比？从实验验证到前瞻性筛查试验，临床转化证据走到了哪一步？

**Findings.** cfDNA 携带核小体占位、片段长度分布、末端基序、甲基化模式等多维信号，单维度特征在多癌种检测中灵敏度通常低于 60%，而五维整合方案（如 CANSCAN）可将 13 种癌症的整体灵敏度提升至 87.4%，I 期灵敏度达 79.3%。甲基化路线（cfMeDIP-seq / 靶向甲基化）与片段化路线（fragmentomics）各有优势，前者组织溯源能力更强，后者实验流程更轻量。NHS-Galleri 等前瞻性 RCT 正在验证 MCED 检测能否转化为死亡率下降。

**Conclusion.** cfDNA 多癌种早检已从概念验证走向前瞻性临床试验阶段，但低肿瘤分数下的灵敏度瓶颈、跨队列泛化性、以及检出到临床获益之间的 evidence gap 仍是落地的核心障碍。

---

## 1. 引言

### 1.1 癌症早筛的临床需求

癌症是全球第二大死因。2022 年 GLOBOCAN 数据显示全球新发病例约 1930 万例、死亡约 1000 万例。一个反复被验证的规律是：确诊越早，预后越好。以肺癌为例，I 期非小细胞肺癌的 5 年生存率约 68%-92%，而 IV 期低于 10%（SEER 2024）。胰腺导管腺癌（PDAC）在可切除阶段确诊的比例不足 20%，多数患者确诊即失去手术机会（Yin, 2026）。

现有筛查工具存在明确的覆盖盲区。低剂量螺旋 CT 仅针对肺癌高危人群，结肠镜覆盖结直肠癌但依从性不足 50%，胃癌、胰腺癌、食管癌等高致死癌种缺乏被指南推荐的筛查手段。一项能从单管血液中同时检出多种癌症信号的检测——多癌种早期检测（multi-cancer early detection, MCED）——因此成为过去十年液体活检领域的核心追求。

### 1.2 cfDNA 作为液体活检标志物的生物学基础

循环游离 DNA 是血浆中以游离态存在的短片段 DNA，主要来源于细胞凋亡和坏死时核小体包裹 DNA 的释放。健康个体的 cfDNA 以造血系统来源为主，片段长度呈现以 ~167 bp 为主峰的核小体周期性分布（Snyder, 2016）。肿瘤患者血浆中混入来自肿瘤细胞的循环肿瘤 DNA（ctDNA），其比例称为肿瘤分数（tumor fraction, TF）。早期癌症患者的 TF 通常极低（中位 <0.5%），这是 cfDNA 早筛面临的根本性挑战。

cfDNA 携带的信息远不止碱基序列。2016 年 Snyder 等人的开创性工作证明，对 cfDNA 进行 96-105x 深度测序可以绘制体内核小体占位图谱（12.9 million peaks），短片段中保留了转录因子足迹信息，且核小体间距模式可推断组织来源——无需依赖遗传学差异（Snyder, 2016）。这一发现将 cfDNA 从"突变载体"重新定位为"表观基因组探针"，为后续 fragmentomics 和甲基化两条技术路线奠定了基础。

### 1.3 综述边界

本综述纳入 2016-2026 年间发表的 33 篇研究，涵盖 cfDNA 分子信号发现、检测方法开发、单癌种验证和多癌种平台临床转化。文献来源以 Zotero 本地库「早筛」collection（52 篇）为主体，按相关度和代表性筛选。排除标准：(1) 非 cfDNA 的液体活检标志物（如 CTC、外泌体蛋白）；(2) 治疗监测/MRD 场景（除非方法可迁移至早筛）；(3) PRISMA-grade 系统综述流程不在本工作范围。

### 1.4 综述结构

本文按五个主题组织。第 2 节梳理 cfDNA 携带的早筛分子信号基础；第 3 节比较不同检测技术路线；第 4 节综述单癌种早筛验证结果；第 5 节聚焦 MCED 平台与临床转化；第 6 节讨论当前挑战和未来方向。

---

## 2. cfDNA 携带的早筛分子信号

### 2.1 核小体占位与组织溯源

cfDNA 并非随机降解的产物。Snyder (2016) 通过对健康人和癌症患者血浆 cfDNA 进行超深度测序（1.5-1.6 billion fragments, 96-105x coverage），建立了基于窗口化保护评分（Windowed Protection Score, WPS）的体内核小体占位图谱。该图谱揭示了三个关键事实：(1) cfDNA 核小体占位与基因结构、核构型和基因表达水平显著相关；(2) 短片段（<120 bp）中保留了转录因子的结合足迹；(3) 健康人 cfDNA 的核小体间距模式与淋巴/髓系细胞最为一致，反映了其造血系统来源。

Doebley (2022) 在此基础上开发了 Griffin 框架，将核小体足迹分析从实验室发现推向标准化工具。Griffin 可从低覆盖度 WGS 数据中提取转录因子结合位点周围的核小体占位模式，用于癌症亚型分类。Lai (2024) 进一步将分析聚焦于调控活性染色质区域，提出从 cfDNA 片段化模式中提取调控元件活性的方法，将核小体足迹的分辨率从基因级提升至调控元件级。

这条从 Snyder (2016) 到 Griffin (2022) 再到调控足迹 (2024) 的发展线索表明，cfDNA 中的核小体信息远未被充分挖掘——每一代方法都在同样的 WGS 数据中提取出更细粒度的生物学信号。

### 2.2 片段长度与方向性特征

cfDNA 片段长度分布是最早被利用的 fragmentomics 特征之一。肿瘤来源的 cfDNA 片段系统性短于正常背景，这种长度偏移在不同基因组区域并不均匀。Sun (2019) 首次报告了 cfDNA 片段化的方向性特征（orientation-aware fragmentation）：在开放染色质区域，cfDNA 片段的 5' 和 3' 端切割位点呈现与核小体定位相关的方向偏好。这种方向性信号为区分不同染色质状态提供了额外维度。

Wang (2025) 提出了片段离散度指数（Fragment Dispersity Index, FDI），量化特定基因组窗口内 cfDNA 片段长度的分散程度作为染色质可及性的代理指标。FDI 的设计思路是：开放染色质区域因核酶切割位点的多样性而产生更离散的片段长度分布，这种离散度本身就携带组织特异性信息。

> **注意**：片段长度类特征（FSD、FSC、FDI）的信噪比高度依赖 TF。当 TF 降至 0.1% 以下时，正常造血来源的 cfDNA 主导了长度分布，肿瘤相关的偏移可能被淹没。这一固有局限促使研究者将片段长度与其他维度信号整合使用，而非单独作为分类器。

### 2.3 甲基化信号

DNA 甲基化模式具有高度的组织特异性，且在肿瘤中发生全基因组水平的重编程。Shen (2018) 开发的 cfMeDIP-seq 方法利用 5-methylcytosine 抗体免疫沉淀替代亚硫酸氢盐转化，仅需 1-10 ng cfDNA 输入即可在全基因组范围内检测甲基化区域。在 7 种癌症的分类任务中，cfMeDIP-seq 达到 AUROC 0.918-0.980（验证集），且早期和晚期样本的灵敏度接近（Shen, 2018）。一项关键发现是：cfDNA 中的差异甲基化区域（DMR）与配对肿瘤组织的甲基化图谱高度重叠（P<1e-745），直接证实了 ctDNA 的组织来源。

Loyfer (2023) 构建了涵盖正常人体细胞类型的 DNA 甲基化图谱（methylation atlas），为 cfDNA 甲基化的组织溯源提供了参考数据集。有了正常组织的甲基化基线，异常甲基化信号的来源推断变得更加可靠。

Zhou (2022a) 从另一角度切入，利用 cfDNA 甲基化模式反推肿瘤分数（tumor fraction deconvolution）。这一思路的优势在于：甲基化变化在肿瘤基因组中的分布极为广泛（涉及数千个 DMR），即使 ctDNA 占比极低，足够数量的 DMR 累积仍有望提供检测信号——这是甲基化路线在低 TF 场景下的理论优势。

> **甲基化路线与 fragmentomics 路线之间存在本质性的方法学张力**。cfMeDIP-seq 通过免疫沉淀步骤富集甲基化 DNA，DMR 数量大，灵敏度优势明显，但需要额外的实验步骤且成本更高。Fragmentomics 路线则直接从 WGS 数据中提取末端基序（end motif）间接推断甲基化状态，实验流程更轻量，且 WGS 数据可持续更新模型而无需对样本重新测序（Bao, 2025）。两条路线在 MCED 领域的竞争与互补将在第 3 节详细讨论。

### 2.4 其他信号维度

除核小体足迹、片段长度和甲基化外，cfDNA 还携带多种可被利用的信号。

**末端基序（end motif）**。cfDNA 片段的 5' 端 4-mer 碱基偏好反映了核酸酶的切割特异性和局部染色质环境。末端基序谱（end motif profile）的异常可作为癌症信号，且可间接推断甲基化状态——因为甲基化的 CpG 位点影响核酸酶的切割偏好。

**基因组缺失序列（neomers）**。Georgakopoulos-Soares (2021) 提出了一个独特视角：健康人基因组中天然不存在的 13-17 nt 短序列（neomers），可能因肿瘤突变而在 cfDNA 中出现。在 2577 例肿瘤基因组和 465 例 cfDNA WGS 的分析中，neomers 在肺癌和卵巢癌中达到 AUC 0.89-0.94（包括早期病例）。这条路线的独特之处在于它不依赖肿瘤与正常信号的"量"差异，而是利用"有/无"的二值判断，理论上可绕过 TF 的限制。

**线粒体 cfDNA（ccf-mtDNA）**。Liu (2024) 首次系统报告了循环游离线粒体 DNA 的片段化特征。在 1607 例血浆样本中，ccf-mtDNA 的片段化模式与核 cfDNA 显著不同，且与蛋白结合、碱基组成和 mtDNA 结构相关。6 种癌症类型均检出异常 mtDNA 片段化特征，检测 AUC 均 >0.93，组织溯源准确率达 87.9%-89.2%（Liu, 2024）。ccf-mtDNA 作为核 cfDNA 之外的独立信号源，可能为多维整合模型提供增量信息。

### 2.5 跨信号 synthesis：单维度 vs 多维度整合

上述分子信号各有其检测窗口和局限。片段长度在高 TF 样本中区分度好但低 TF 下信噪比骤降；甲基化 DMR 数量大但需额外实验步骤；核小体足迹分辨率高但依赖深度测序；neomers 不受 TF 限制但覆盖的突变类型有限。

单维度 fragmentomics 方法在多癌种场景下的整体灵敏度通常低于 60%（Bao, 2025）。CANSCAN 平台通过整合五个维度——拷贝数变异（CNV）、片段大小覆盖度（FSC）、片段大小分布（FSD）、核小体足迹（NP）和末端基序（FM）——将 13 种癌症的整体灵敏度提升至 87.4%（特异性 97.8%），I 期灵敏度达到 79.3%，远超既往报告的 27.5%-43.8%（Bao, 2025）。

这一数据差距传递了一个清晰信息：cfDNA 早筛的突破不在于发现单一"超级特征"，而在于从同一份 WGS 数据中系统提取并整合多个正交的信号维度。

---

## 3. 检测策略与技术路线

### 3.1 Fragmentomics 多维整合路线

Fragmentomics 路线的核心策略是从低覆盖度全基因组测序（WGS）数据中提取多种片段化特征，通过机器学习模型整合分类。

Zhou (2022b) 率先证明 cfDNA 片段化模式可用于推断表观遗传学信息（epigenetic analysis by fragmentomic profiling），将 fragmentomics 从物理特征分析扩展到表观基因组推断。He (2024) 进一步开发了基于片段的泛癌差异甲基化标记物发现算法，在不进行甲基化特异性实验的前提下，从 WGS 片段模式中识别甲基化异常区域。

这一技术路线的集大成者是 Bao (2025) 报告的 CANSCAN 平台。CANSCAN 从单次低覆盖度 WGS 中同时提取 5 类特征（CNV、FSC、FSD、NP、FM），每类特征先由独立的机器学习模型打分，再通过 stacked ensemble 策略整合为最终的癌症检测和组织溯源评分。在涵盖 13 种癌症类型的独立验证集中（SST），整体灵敏度 87.4%、特异性 97.8%；金陵前瞻队列（N=3724）中灵敏度 53.5%（93% 为早期），特异性 98.1%，NPV 99.4%（Bao, 2025）。

> **金陵前瞻队列的灵敏度（53.5%）与回顾性验证集（87.4%）之间存在显著落差**。这并非 CANSCAN 特有的问题——几乎所有 MCED 平台在从 case-control 设计转向真实筛查人群时都观察到灵敏度下降，根本原因在于筛查人群中早期、低 TF 病例的比例远高于回顾性队列。

### 3.2 靶向甲基化路线

甲基化路线以 GRAIL/Galleri 平台为代表。Liu (2021) 报告了基于靶向甲基化测序的多癌种检测和定位方案，在 CCGA (Circulating Cell-free Genome Atlas) 的前瞻性病例对照子研究中，通过靶向覆盖基因组中信息量最大的甲基化位点，实现高特异性下的多癌种检测和组织溯源。甲基化信号的组织溯源能力是该路线的突出优势——甲基化模式本身就是细胞身份的标志。

Jamshidi (2022) 在 CCGA 子研究中直接对比了三种 cfDNA 检测策略：靶向甲基化、全基因组测序和靶向突变 panel。结果显示靶向甲基化在灵敏度-特异性综合表现上优于其他两种策略，尤其在低 TF 样本中优势更为明显。这一对比为 Galleri 最终选择甲基化路线提供了直接证据。

> **靶向甲基化与 WGS fragmentomics 的取舍并非简单的"哪个更好"**。靶向甲基化测序深度更高（因聚焦于信息量大的位点），在低 TF 下的单分子检测能力更强；但其覆盖的信息维度有限（仅甲基化），且需要针对特定位点设计 panel——一旦 panel 确定，无法回溯性地提取新发现的信号类型。Fragmentomics 路线的 WGS 数据则可持续重新分析以纳入新特征，且无需甲基化特异性的实验步骤。两条路线在灵敏度、可扩展性和成本之间形成互补而非替代关系。

### 3.3 AI 驱动路线

传统 cfDNA 分析依赖人工设计的特征（如 fragment size ratio, end motif frequency），AI 驱动路线则尝试直接从原始测序数据中端到端学习检测信号。

Shen (2024) 开发的 ACID（Affordable Cancer Interception and Diagnostics）模型是这一方向的代表。ACID 基于 DNA 语言模型架构，直接以 cfDNA 测序 reads 作为输入，无需预先定义特征工程步骤，在测试集上 AUROC 达 0.924（显著优于基准方法的 0.853, P<0.001），仅需每样本 10,000 条 reads 即可达到高准确度（Shen, 2024）。Li (2021) 的 Dismir 则整合了 DNA 序列和甲基化两种信息，通过深度学习模型在 cfDNA 全基因组亚硫酸氢盐测序数据中实现单 read 级别的来源预测，在肝细胞癌检测中即使在超低测序深度下仍保持高准确性和鲁棒性（Li, 2021）。Liu (2023) 采用另一策略，直接从原始测序片段进行癌症诊断，绕过比对和特征提取步骤。

Tsui (2025) 在一篇综述中系统梳理了 AI/ML 在 cfDNA 诊断中的应用，指出当前 AI 模型面临的核心挑战：(1) 训练数据的规模和多样性不足导致过拟合风险；(2) 端到端模型的可解释性差，难以获得监管认可；(3) 不同 AI 方法之间缺乏标准化的性能比较框架。

> **AI 路线的 promise 与 reality 之间存在张力**。端到端模型在封闭数据集上的表现往往优于传统特征工程方法，但在跨平台、跨人群部署时是否保持优势尚未充分验证。Van Calster (2025) 对预测性 AI 模型的评估方法提出了系统性建议，强调外部验证的样本量、人群分布和预设性能阈值对于评估结果可靠性的决定性影响——这些标准同样适用于 cfDNA 领域的 AI 模型。

### 3.4 方法学比较

| 特征 | Fragmentomics 多维整合 | 靶向甲基化 | AI 端到端 |
|------|----------------------|-----------|----------|
| **代表平台** | CANSCAN (Bao, 2025) | Galleri (Liu, 2021) | ACID (Shen, 2024) |
| **测序策略** | 低覆盖度 WGS (~5x) | 靶向 bisulfite | 低覆盖度 WGS |
| **输入要求** | 5-10 ng cfDNA | 30-50 ng cfDNA | 5-10 ng cfDNA |
| **特征提取** | 人工设计 5 维 | 甲基化位点覆盖 | 端到端学习 |
| **多癌种灵敏度** | 87.4% (SST) | 待 NHS-Galleri 公布 | AUROC 0.924 (测试集) |
| **I 期灵敏度** | 79.3% | 27.5%-43.8% (CCGA) | 未充分验证 |
| **组织溯源** | TOP1 82.4% | TOP1 ~89% | 架构内可输出 |
| **可扩展性** | 高（WGS 可重分析） | 低（panel 固定） | 高（重训练） |
| **可解释性** | 中（特征可追溯） | 高（DMR 可定位） | 低 |
| **额外实验步骤** | 无 | bisulfite 转化或 IP | 无 |

### 3.5 路线之争：fragmentomics vs methylation 的互补与竞争

cfDNA 早筛领域当前形成了两条主要技术路线的格局：以 Galleri 为代表的甲基化路线和以 CANSCAN 为代表的 fragmentomics 多维整合路线。两者的竞争不仅是技术选择，也是商业化策略的分歧。

甲基化路线的优势在于 DMR 数量庞大（每癌种可达数千个），即使在极低 TF 下仍有理论检测窗口（Shen, 2018）；且甲基化模式的组织特异性高，天然适合组织溯源。其代价是需要额外的甲基化富集或转化步骤，cfDNA 输入量更高，成本更高。

Fragmentomics 路线的优势在于：(1) 从单次低覆盖度 WGS 中提取多维信号，无需额外实验步骤；(2) WGS 数据可回溯性地分析新发现的特征，模型可迭代升级而无需对样本重新测序；(3) 对 cfDNA 输入量要求更低。CANSCAN 的 I 期灵敏度（79.3%）相较既往甲基化方案（27.5%-43.8%）的跃升，说明多维整合在早期检测中的价值。但需注意两组数据来自不同队列和时期，直接比较需谨慎。

未来的趋势可能不是二选一，而是融合。Zhou (2022b) 和 He (2024) 的工作已经展示了从 fragmentomics 数据中推断甲基化信息的可能性——即"不做甲基化实验，但获得甲基化维度的信号"。如果这条技术路径成熟，fragmentomics 和 methylation 的边界可能逐渐模糊。

---

## 4. 单癌种早筛验证

MCED 的可行性建立在各癌种 cfDNA 信号可被独立检出这一前提之上。本节按癌种归类综述代表性验证研究，重点关注灵敏度、分期分层表现和方法学差异。

### 4.1 消化道癌症

**结直肠癌（CRC）** 是 cfDNA 早筛验证最为充分的癌种之一。Cao (2024) 采用 5 类 fragmentomics 特征构建 ensemble stacked 模型，训练集 360 例（176 CRC + 184 对照），独立验证集 AUROC 0.986，灵敏度 94.88%，特异性 98%；在前瞻性队列中灵敏度 91.47%，特异性 95.58%（Cao, 2024）。Xie (2024) 从甲基化路线切入，开发了基于甲基化扩增探针（methylation amplifier probe）的超灵敏 ctDNA 检测方法，在 CRC 先导研究中验证了其可行性。两项工作从不同技术路线达到了接近的性能水平，提示 CRC 的 cfDNA 信号释放量相对充足，是 MCED 检测"容易"的癌种。

**胃癌**的 cfDNA 检测面临更大挑战——早期胃癌的 TF 低于 CRC，且胃癌的分子异质性高。Yu (2024) 整合了 4 种 cfDNA 特征（片段大小、CNV、核小体足迹、SNV）构建 ensemble 模型，在研究队列中 AUROC 0.962，灵敏度 88.2%（110 例 I-II 期），特异性 92.1%；第一验证集 AUROC 0.972，灵敏度 91.8%；第二验证集 AUROC 0.937，灵敏度 87.2%（Yu, 2024）。跨验证集的一致性（AUROC 0.937-0.972）为该方法的稳健性提供了初步证据。

**胰腺导管腺癌（PDAC）** 是早筛需求最迫切但难度最大的癌种。PDAC 患者在可切除阶段确诊的比例不足 20%，多数确诊即 IV 期。Yin (2026) 开发了基于 cfDNA fragmentomics 整合高级机器学习的 PDAC 早期检测模型，初步结果显示在早期 PDAC 中具有较高的检测准确性（Yin, 2026）。但 PDAC 的特殊困难在于：肿瘤体积小、基质丰富导致 ctDNA 释放极少，且胰腺位于腹膜后，缺乏解剖学上的"天然出口"——这些因素共同造成 PDAC 在所有 MCED 平台中灵敏度普遍最低。

**食管癌**。Cheng (2025) 采用 advanced ensemble stacking 策略整合 cfDNA 片段化特征，在食管癌早期检测中取得了可行性验证结果（Cheng, 2025）。食管鳞癌在中国高发区域的筛查需求使得基于 cfDNA 的无创方案具有特殊的公共卫生价值。

### 4.2 妇科肿瘤

**子宫内膜癌（EC）**。Rao (2025) 报告的 DECIPHER-UCEC-2 研究纳入 120 例 EC 和 120 例健康对照，从低覆盖度 WGS 提取 5 类 fragmentomic 特征并整合 4 种机器学习算法。独立测试集 AUC 0.96，灵敏度 75.8%，特异性 96.8%。按分期看，I-IV 期灵敏度分别为 74.4%、85.7%、75%、75%（Rao, 2025）。

该研究的一个额外发现值得关注：同一模型框架在亚型分类（组织学亚型 AUC 0.73、MSI 状态 AUC 0.77）和预后预测（无复发生存 HR=8.6, P<0.001）上也表现出潜力。如果同一次采血同时完成检测、分型和预后分层，cfDNA 早筛的临床价值将从"发现癌症"扩展到"指导管理"。

### 4.3 肺癌

Guo (2022) 针对 I 期侵袭性肺腺癌（LUAD），利用 cfDNA 断点基序谱（breakpoint motif profiling）构建了检测模型，在 I 期 LUAD 中的灵敏度优于传统的 cfDNA 突变检测方案。断点基序是一种独特的 fragmentomics 信号——它不仅反映核酸酶切割偏好，还可能与肿瘤特异性的 DNA 修复缺陷相关（Guo, 2022）。

> **肺癌早筛的特殊语境**：低剂量 CT（LDCT）已被多国指南推荐为肺癌高危人群筛查工具。cfDNA 在肺癌场景下更可能作为 LDCT 的补充而非替代——DECIPHER-NODL 的数据显示，cfDNA 在纯实性结节中表现更优，LDCT 在亚实性结节中更优，二者互补可将特异性从 0.33-0.50 提升至 0.60（Bao, 2025）。

### 4.4 跨癌种比较：信号强度不均等

综合上述验证结果可以归纳出一个模式：cfDNA 信号强度在不同癌种间显著不均等。

| 癌种 | 代表性灵敏度 | 信号强度 | 可能原因 |
|------|------------|---------|---------|
| CRC | 91-95% | 强 | 肠道黏膜更新快，ctDNA 释放量大 |
| 胃癌 | 87-92% | 中-强 | 黏膜来源，但异质性高 |
| 肝癌 | >89% (TOP1) | 强 | 肝脏血流丰富，ctDNA 清除快但释放也快 |
| 肺癌 (I期) | ~75-80% | 中 | I 期肿瘤体积小，但可借助 LDCT 互补 |
| EC | 74-76% (I期) | 中 | 子宫内膜肿瘤 ctDNA 释放受解剖位置限制 |
| PDAC | 最低 | 弱 | 肿瘤体积小、基质丰富、腹膜后位置 |
| 胰腺/胃 TOO | 45-50% | -- | 组织溯源最困难的两个癌种 |

这种不均等意味着：MCED 测试的"一管血测多癌"承诺在不同癌种上的兑现程度不同。对于信号弱的癌种（PDAC、部分妇科肿瘤），可能需要癌种特异性的增敏策略或与其他筛查手段联合使用。

---

## 5. 多癌种早检平台与临床转化

### 5.1 MCED 平台演进

MCED 概念的实验验证可追溯到 Cohen (2018) 发表的 CancerSEEK。CancerSEEK 采用多分析物策略（cfDNA 突变 + 蛋白标志物），在 8 种癌症（卵巢、肝、胃、胰腺、食管、结直肠、肺、乳腺）的 1005 例确诊患者中检测，中位灵敏度 70%，特异性 >99%（Cohen, 2018）。CancerSEEK 的价值不在于其绝对性能——按今天的标准看，多个癌种的灵敏度不足——而在于它首次证明了"一管血检多癌"的可行性，并提出了组织溯源作为 MCED 测试的必要功能。

此后，GRAIL 公司基于 CCGA 大队列开发了以靶向甲基化为核心的 Galleri 检测。Liu (2021) 报告的 CCGA 子研究确认了甲基化路线在多癌种检测和组织溯源上的能力。Jamshidi (2022) 在 CCGA 中对比了三种 cfDNA 策略（靶向甲基化、全基因组测序、靶向突变），结果明确支持靶向甲基化在灵敏度-特异性综合表现上的优势。Galleri 成为第一个进入大规模 RCT 的 MCED 检测（详见 5.3 节）。

CANSCAN (Bao, 2025) 代表了 fragmentomics 多维整合路线的最新进展。相较 Galleri 的甲基化路线，CANSCAN 的差异化在于：(1) 覆盖 13 种癌症（占全球 66.6% 新发和 74.0% 死亡）；(2) I 期灵敏度 79.3% 远超甲基化方案的既往报告（27.5%-43.8%）；(3) 基于低覆盖度 WGS，无需甲基化特异性实验。但需注意 CANSCAN 与 Galleri 的数据来自不同队列，且 Galleri 的最新前瞻性数据尚未完整公布——直接比较两个数字需要审慎。

### 5.2 组织溯源的临床价值

MCED 检测阳性后，临床医生面临一个直接问题："癌在哪里？"组织溯源（tissue-of-origin, TOO）的准确性决定了后续诊断流程的效率。

CANSCAN 报告的 TOO 准确率为 TOP1 82.4%、TOP2 91.7%（Bao, 2025）。分癌种看，结直肠癌、肺癌、肝癌和淋巴瘤的 TOP1 准确率 >89%，而胰腺癌（50.0%）和胃癌（45.7%）最低（Bao, 2025）。

> **CANSCAN 的 TOO 准确率（82.4%）低于 Galleri 报告的 88.7%。** 但两组数据的上下文不同：CANSCAN 的灵敏度更高，纳入了更多低信号患者——这些患者本身的 cfDNA 信号弱，溯源难度更大。高灵敏度与高 TOO 准确率之间存在内在张力：检出越多低信号病例，整体 TOO 准确率越可能被拉低。评估 MCED 平台时，灵敏度和 TOO 需作为一个整体看，而非分别比较。

### 5.3 前瞻性验证：从 case-control 到 RCT

MCED 领域最关键的临床转化里程碑是 NHS-Galleri 试验（ISRCTN91431511）。Neal (2022) 报告了该试验的设计：在英格兰一般人群中（50-77 岁），约 150 万人受邀、超过 14 万人入组，1:1 随机分配至干预组（接受 Galleri MCED 检测）和对照组（采血储存），每年检测一次共 3 年，主要终点为随机化后 3-4 年内 III/IV 期癌症发病率的下降幅度（Neal, 2022）。

NHS-Galleri 是全球首个以"stage shift"（即期别前移）为主要终点的 MCED RCT。这一设计回应了 MCED 领域最核心的批评：**检出更多早期癌症不等于降低死亡率**。只有当早期检出确实带来了更多治愈性治疗机会，且避免了过度诊断带来的净伤害时，MCED 才有公共卫生价值。

金陵前瞻队列（PROMOTE, N=3724）为 CANSCAN 提供了前瞻性验证数据：灵敏度 53.5%（93% 为早期），特异性 98.1%，NPV 99.4%（Bao, 2025）。PPV 为 25%，低于 PATHFINDER 试验报告的 38%。

> **PPV 差异需要在队列设计背景下理解。** 金陵队列以标准体检确认最终诊断，而 PATHFINDER 以 MCED 阳性触发诊断流程——后者设计上倾向于更快确认真阳性，导致 PPV 偏高。PPV 的绝对值高度依赖癌症患病率和诊断确认流程，不适合跨队列直接比较。

### 5.4 卫生经济学证据

Sasieni (2023) 对 MCED 在英格兰筛查人群中的死亡率获益进行了模型预测。该模型基于 NHS-Galleri 的队列参数和 Galleri 已公布的灵敏度数据，估算了 MCED 检测如果实现"stage shift"后可能带来的癌症死亡率下降幅度。在最保守的假设下，每 10 万筛查人群每年可减少约 74 例（17%）癌症死亡（Sasieni, 2023）。建模结果支持 MCED 在人群水平上具有降低死亡率的潜力，但模型的关键假设——检测灵敏度在真实筛查人群中与回顾性研究一致——尚待 NHS-Galleri RCT 验证。

### 5.5 中国场景

2025 年发布的中国液体活检多癌早筛专家共识针对中国人群的癌谱特点（胃癌、食管癌、肝癌高发）、医疗资源分布和支付体系，提出了 MCED 临床应用的推荐框架。中国场景的特殊性在于：(1) 高发癌种与欧美不同，需要适配的检测 panel；(2) 基层医疗机构的后续确诊能力参差不齐——MCED 阳性后的诊断流程是否能有效运转，直接影响筛查的净获益；(3) 国产 MCED 平台（如 CANSCAN）已产出本土前瞻性数据，但尚需更大规模的卫生经济学评估。

---

## 6. 挑战与未来方向

### 6.1 低肿瘤分数下的灵敏度瓶颈

早期癌症患者的 TF 中位值通常 <0.5%，部分 I 期肿瘤低于 0.1%。在这一区间，所有 cfDNA 信号类型的信噪比急剧下降。CANSCAN 在回顾性验证中 I 期灵敏度达 79.3%，但前瞻性队列中整体灵敏度降至 53.5%——差距的主要来源正是前瞻性队列中低 TF 病例比例更高。

**可能的突破路径**：(1) ccf-mtDNA 作为核 cfDNA 的补充信号源，其每细胞拷贝数远高于核 DNA（数百至数千拷贝），理论上在低 TF 下仍可提供检测窗口（Liu, 2024）；(2) 更长的测序 reads 或长读长技术可能捕获更完整的表观遗传信息；(3) 多次纵向采样替代单次检测，利用时间序列信息降低假阴性。

### 6.2 跨队列泛化性

Su (2023) 直接检验了 cfDNA fragmentomics 特征的跨研究泛化性，这是该领域少有的专门面向可重复性问题的工作。核心担忧是：在一个中心、一个平台、一组样本上训练的模型，在不同中心的独立样本上能否保持性能？

泛化性问题的根源多样：测序平台差异（PCR-free vs PCR-based 建库）、cfDNA 提取方法不同、样本采集和储存条件变异、以及训练队列与目标人群在年龄/性别/种族/伴随疾病上的分布差异。这些技术和流行病学混杂因素可能系统性地改变 cfDNA fragmentomics 特征，导致模型在跨场景部署时性能衰减。

### 6.3 AI 模型的可解释性与评估标准

端到端 AI 模型（如 ACID）在特征工程方面的自由度更高，但代价是可解释性下降。Van Calster (2025) 对预测性 AI 模型的评估提出了系统性建议，强调三个要素：(1) 外部验证须在时间和地域上独立于开发集；(2) 性能报告须包括校准度（calibration）而非仅区分度（discrimination）；(3) 临床效用须通过决策曲线分析（DCA）而非仅 AUROC 评估。

这些标准在 cfDNA 早筛领域的执行仍不充分。多数研究仅报告 AUROC 和灵敏度/特异性，缺乏校准度评估和 DCA。监管机构（如 FDA、NMPA）对 AI 驱动的体外诊断产品的审批标准仍在演变中，可解释性要求可能成为端到端模型进入临床的瓶颈。

### 6.4 从检测到临床获益的 evidence gap

MCED 检测能力的提升并不自动等同于临床获益。"检出更多早期癌症"与"降低癌症死亡率"之间至少存在三个 gap：

1. **过度诊断风险**：某些惰性肿瘤（如低级别前列腺癌、部分甲状腺癌）即使不治疗也不影响寿命。MCED 如果将这些病例检出，可能导致不必要的手术和治疗，净效果为伤害而非获益。

2. **确诊流程瓶颈**：MCED 阳性后需要影像学确认和组织活检。如果医疗系统不具备高效的后续确诊能力（尤其在资源有限的基层机构），阳性结果可能带来焦虑而非及时治疗。金陵前瞻队列 PPV 25% 意味着每 4 个阳性中 3 个是假阳性——这些人的心理负担和不必要的后续检查成本不容忽视。

3. **治疗可及性**：早期检出只有在患者能获得有效治疗时才有意义。在治疗手段有限的癌种（如 PDAC）中，早期检出是否改变预后仍有争议。

NHS-Galleri 试验的终极价值正在于回答这一问题：MCED 检测是否能在人群水平上实现 stage shift 并最终降低死亡率？在 RCT 结果公布之前，这仍是一个开放问题。

### 6.5 未来方向

1. **多组学整合与多时间点纵向监测**。现有 MCED 平台多为单次采血、单组学检测。将 cfDNA fragmentomics 与 cfDNA 甲基化、cfRNA、蛋白标志物整合（如 CancerSEEK 的思路但在更高技术水平上重现），并结合多次纵向采样以捕获动态变化，可能是突破低 TF 灵敏度瓶颈的路径。具体建议：设计包含基线和 6-12 月随访采样的前瞻性队列，评估纵向 delta 信号的增量价值。

2. **cfDNA 与影像学的系统性联合策略**。DECIPHER-NODL 的数据已提示 cfDNA 与 LDCT 在不同结节类型上的互补性。未来需要在更多癌种（如胃镜 + cfDNA 联合筛查胃癌、cfDNA + 超声联合筛查甲状腺癌）上验证联合策略的净获益，并建立联合判读的标准化流程。具体建议：在胃癌高发区域开展 cfDNA + 内镜联合筛查的随机对照试验。

3. **标准化与质控体系建设**。cfDNA 早筛从研究走向临床的一个基础性缺口是缺乏标准化的样本处理、测序和分析流程。不同实验室之间的 cfDNA 提取方法、建库方案和生物信息学 pipeline 差异可能导致结果不可比。具体建议：由行业联盟（如中国液体活检共识专家组）牵头制定 cfDNA 早筛的标准参考品和质控方案，建立多中心比对数据集。

---

## 7. 结论

基于 cfDNA 的多癌种早期检测在过去十年中完成了从概念验证到前瞻性临床试验的跨越。cfDNA 携带的核小体占位、片段长度分布、末端基序和甲基化模式构成了丰富的信号空间，多维整合策略（如 CANSCAN 的 5 维 fragmentomics）已在回顾性队列中将 13 种癌症的 I 期灵敏度提升至约 80%。甲基化路线和 fragmentomics 路线各有优势，融合趋势已经显现。NHS-Galleri 等大规模 RCT 正在检验这些检测能力能否转化为死亡率下降的终极证据。

当前的核心未解决问题是：在真实筛查人群中（而非回顾性 case-control 队列），MCED 检测的灵敏度是否足以实现有临床意义的 stage shift，且这一 shift 产生的净获益是否大于过度诊断和假阳性带来的伤害？在这一问题得到前瞻性 RCT 数据回答之前，cfDNA 多癌种早筛仍处于"极具潜力但尚未完全验证"的阶段。

---

## References

1. Snyder MW, Kircher M, Hill AJ, Daza RM, Shendure J. Cell-free DNA comprises an in vivo nucleosome footprint that informs its tissues-of-origin. *Cell*. 2016;164(1-2):57-68. doi:10.1016/j.cell.2015.11.050
2. Sun K, Jiang P, Cheng SH, et al. Orientation-aware plasma cell-free DNA fragmentation analysis in open chromatin regions informs tissue of origin. *Genome Res*. 2019;29(3):418-427. doi:10.1101/gr.242719.118
3. Shen SY, Singhania R, Fehringer G, et al. Sensitive tumour detection and classification using plasma cell-free DNA methylomes. *Nature*. 2018;563(7732):579-583. doi:10.1038/s41586-018-0703-0
4. Loyfer N, Magenheim J, Peretz A, et al. A DNA methylation atlas of normal human cell types. *Nature*. 2023;613(7943):355-364. doi:10.1038/s41586-022-05580-6
5. Lai D, Dilger A, Cunningham S, et al. Extracting regulatory active chromatin footprint from cell-free DNA. *Nucleic Acids Res*. 2024;52(17):e80. [DOI 未验证]
6. Wang Y, Hou Y, Xiong Z, et al. Fragment dispersity index analysis of cfDNA fragments reveals chromatin accessibility and enables early cancer detection. *Cell Rep Methods*. 2025;5(7):101083. doi:10.1016/j.crmeth.2025.101083
7. Georgakopoulos-Soares I, Yizhar-Barnea O, Mouratidis I, et al. Leveraging sequences missing from the human genome to diagnose cancer. *Nat Commun*. 2021;12:4884. doi:10.1101/2021.08.15.21261805 [Preprint; journal DOI pending]
8. Liu H, et al. Aberrant fragmentomic features of circulating cell-free mitochondrial DNA as novel biomarkers for multi-cancer detection. *Clin Chem*. 2024;70(10):1252-1263. [DOI 未验证]
9. Zhou X, Cheng Z, Dong M, et al. Epigenetic analysis of cell-free DNA by fragmentomic profiling. *Proc Natl Acad Sci USA*. 2022;119(44):e2209944119. [DOI 未验证]
10. He X, et al. A fragment-based algorithm for identifying pan-cancer differential methylation markers from cell-free DNA. *J Clin Oncol*. 2024;42(16_suppl):e22507. doi:10.1200/jco.2024.42.16_suppl.e22507
11. Zhou X, Cheng Z, Dong M, et al. Tumor fractions deciphered from circulating cell-free DNA methylation for cancer early diagnosis. *Nat Commun*. 2022;13:7694. doi:10.1038/s41467-022-35320-3
12. Doebley AL, Ko M, Liao H, et al. A framework for clinical cancer subtyping from nucleosome profiling of cell-free DNA. *Nat Commun*. 2022;13:7475. [DOI 未验证]
13. Shen H, Liu J, Chen K, Li X. Language model enables end-to-end accurate detection of cancer from cell-free DNA. *Brief Bioinform*. 2024;25(2):bbae053. doi:10.1093/bib/bbae053
14. Li J, Wei L, Zhang X, et al. DISMIR: Deep learning-based noninvasive cancer detection by integrating DNA sequence and methylation information of individual cell-free DNA reads. *Brief Bioinform*. 2021;22(6):bbab250. doi:10.1093/bib/bbab250
15. Liu Y, et al. A deep learning approach for cancer diagnosis exclusively from raw sequencing fragments of cell-free DNA. *Nat Commun*. 2023;14:4830. [DOI 未验证]
16. Tsui NB, et al. Artificial intelligence and machine learning in cell-free-DNA-based diagnostics. *Nat Rev Clin Oncol*. 2025. [DOI 未验证]
17. Cao Y, Wang N, Wu X, et al. Multidimensional fragmentomics enables early and accurate detection of colorectal cancer. *Cancer Res*. 2024;84(19):3139-3152. doi:10.1158/0008-5472.CAN-24-1620
18. Xie H, et al. A pilot study on ultra-sensitive circulating tumor DNA detection with a novel methylation amplifier probe in colorectal cancer. *Clin Chem Lab Med*. 2024;62(6):1159-1168. [DOI 未验证]
19. Yu P, Chen P, Wu M, et al. Multi-dimensional cell-free DNA-based liquid biopsy for sensitive early detection of gastric cancer. *Mol Cancer*. 2024;23:120. [DOI 未验证]
20. Yin X, et al. Development and validation of a cell-free DNA fragmentomics-based model for early detection of pancreatic ductal adenocarcinoma. *EBioMedicine*. 2026. [DOI 未验证]
21. Rao M, et al. Early detection, clinicopathological subtyping, and prognosis prediction for endometrial cancer using fragmentomics liquid biopsy assay. *Cell Rep Med*. 2025. [DOI 未验证]
22. Cheng W, et al. Advanced ensemble stacking model employing cfDNA fragmentation for early detection of esophageal cancer. *J Mol Diagn*. 2025. [DOI 未验证]
23. Guo S, et al. Sensitive detection of stage I lung adenocarcinoma using plasma cell-free DNA breakpoint motif profiling. *Cancer Res*. 2022;82(14):2537-2547. [DOI 未验证]
24. Cohen JD, Li L, Wang Y, et al. Detection and localization of surgically resectable cancers with a multi-analyte blood test. *Science*. 2018;359(6378):926-930. doi:10.1126/science.aar3247
25. Liu MC, Oxnard GR, Klein EA, et al. Sensitive and specific multi-cancer detection and localization using methylation signatures in cell-free DNA. *Ann Oncol*. 2020;31(6):745-759. doi:10.1016/j.annonc.2020.02.011
26. Jamshidi A, Liu MC, Klein EA, et al. Evaluation of cell-free DNA approaches for multi-cancer early detection. *Cancer Cell*. 2022;40(12):1537-1549. [DOI 未验证]
27. Neal RD, Johnson P, Clarke CA, et al. Cell-free DNA-based multi-cancer early detection test in an asymptomatic screening population (NHS-Galleri): design of a pragmatic, prospective randomised controlled trial. *Cancers*. 2022;14(19):4818. doi:10.3390/cancers14194818
28. Bao H, Wang Z, Ma X, et al. Early detection of multiple cancer types using multidimensional cell-free DNA fragmentomic features. *Nat Med*. 2025. [DOI 未验证]
29. Sasieni P, Smittenaar R, Hubbell E, et al. Modelled mortality benefits of multi-cancer early detection screening in England. *Br J Cancer*. 2023;129(1):72-80. doi:10.1038/s41416-023-02243-9
30. 中国液体活检多癌种早期筛查专家共识 (2025). [DOI 未验证]
31. Su Y, et al. Testing the generalizability of cfDNA fragmentomic features across different studies for cancer early detection. *Sci Rep*. 2023;13:12643. [DOI 未验证]
32. Van Calster B, et al. Evaluation of performance measures in predictive artificial intelligence models to support clinical practice. *JAMA*. 2025. [DOI 未验证]
33. Bruhm DC, Vulpescu NA, Foda ZH, et al. Genomic and fragmentomic landscapes of cell-free DNA for early cancer detection. *Science*. 2025. [DOI 未验证]
