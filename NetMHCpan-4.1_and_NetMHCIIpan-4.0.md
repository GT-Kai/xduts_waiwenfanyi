# NetMHCpan-4.1 和 NetMHCIIpan-4.0：通过同步基序解卷积并整合 MS MHC 洗脱配体数据，改进 MHC 抗原呈递预测

原文：Birkir Reynisson, Bruno Alvarez, Sinu Paul, Bjoern Peters and Morten Nielsen. *Nucleic Acids Research*, 2020, Vol. 48, Web Server issue, W449-W454. doi: 10.1093/nar/gkaa379  
在线发表：2020 年 5 月 14 日  
收稿：2020 年 3 月 13 日；修回：2020 年 4 月 17 日；编辑决定：2020 年 4 月 29 日；接受：2020 年 4 月 29 日

作者单位：

1. 丹麦技术大学生物与健康信息学系，Kgs. Lyngby，DK 28002，丹麦
2. 圣马丁国立大学生物技术研究所，Buenos Aires，BA 16503，阿根廷
3. La Jolla Institute for Immunology，La Jolla，CA 92037，美国
4. 加利福尼亚大学圣迭戈分校医学院，La Jolla，CA 92093，美国

通讯作者：Morten Nielsen，morni@dtu.dk  
注：前两位作者贡献相同，应视为共同第一作者。

## 摘要

主要组织相容性复合体（major histocompatibility complex，MHC）分子表达于细胞表面，在那里向 T 细胞呈递肽段，因此在 T 细胞免疫应答的形成中发挥关键作用。MHC 分子主要有两类：MHC I 类（MHC-I）和 MHC II 类（MHC-II）。MHC-I 主要呈递来自细胞内蛋白的肽段，而 MHC-II 主要呈递来自细胞外蛋白的肽段。在这两种情况下，MHC 与抗原肽之间的结合都是抗原呈递通路中选择性最强的一步。因此，预测肽段与 MHC 的结合，是推断 T 细胞免疫应答可能特异性的有力工具。

常见的 MHC 结合预测工具通常使用结合亲和力数据，或质谱洗脱配体数据进行训练。然而，近期研究已经表明，将这两类数据整合起来可以提升预测性能。受此启发，本文介绍 NetMHCpan-4.1 和 NetMHCIIpan-4.0 两个网络服务器，分别用于预测肽段与 MHC-I、MHC-II 之间的结合。两种方法都采用专门设计的机器学习策略来整合不同类型的训练数据，从而取得当前领先水平的性能，并优于竞争方法。服务器地址分别为：

- http://www.cbs.dtu.dk/services/NetMHCpan-4.1/
- http://www.cbs.dtu.dk/services/NetMHCIIpan-4.0/

## 引言

主要组织相容性复合体（MHC）是脊椎动物细胞免疫系统中一种基础性的细胞表面蛋白。MHC 的主要功能是结合由细胞内或细胞外蛋白消化产生的肽段（小的蛋白片段），并将它们展示到细胞间空间。如果 T 细胞识别并结合某个肽-MHC 复合体，便可能触发免疫应答，受损细胞会发生裂解。鉴于此，抗原肽与 MHC 分子的结合是细胞免疫的必要步骤；理解这一事件的规律，对于人类健康应用具有巨大且重要的潜力。

MHC 主要有两类：MHC I 类（MHC-I）和 MHC II 类（MHC-II）。MHC-I 结合来自细胞内蛋白的肽段，这些蛋白先经历蛋白酶体降解；MHC-I 因而作为一种控制机制，用于监测自身肽组库中的抗原性变化。另一方面，MHC-II 结合由细胞外蛋白经蛋白酶消化产生的肽段；借助这一机制，两个 MHC 系统都可以通过向 T 细胞呈递非自身蛋白来监控外来生物（1）。基于这一事实，研究者投入了大量努力来开发能够准确预测肽段与 MHC-I 及 MHC-II 结合的计算方法（综述见文献 2）。

不同类型的实验数据已被用于训练这些方法。根据训练数据的性质，肽-MHC 结合预测器可分为三大类。第一类是使用结合亲和力（binding affinity，BA）数据训练的预测器（3-6）。这类数据会对预测性能造成显著限制，因为它只建模肽-MHC 结合这一个事件，而忽略了该过程中涉及的其他生物学特征。

第二类方法要么使用质谱（mass spectrometry，MS）实验获得的数据训练，即所谓洗脱配体（eluted ligand，EL）数据（7-11）；要么同时整合 BA 和 EL 数据进行训练（5,12-15）。后一类数据不仅包含与肽-MHC 结合事件有关的信息，还包含抗原呈递生物学通路中前置步骤的信息。然而，除基因工程细胞外，由于存在多种 MHC 等位基因变体，细胞的 MHC 表达谱非常多样。此外，MS EL 流程中用于纯化肽-MHC 复合体的抗体多为泛特异性或位点特异性，因此得到的数据天然具有多重特异性（或多等位基因，multi-allelic，MA）：也就是说，数据中包含的肽段可匹配多个同源 MHC 结合基序。因此，通常需要预先设定、且带有用户偏向的肽-MHC 注释标准，才能解释这类 EL MA 数据，将其转换为单等位基因数据（EL SA，即单一肽-MHC 注释），并用于训练 MHC 特异性结合预测器（16）。

第三类也是最后一类算法试图解决第二类模型的局限：它在训练预测算法的同时，纳入了将 EL MA 序列注释到单个 MHC 限制性的能力（17,18）。其中一种方法称为 NNAlign MA（17）。在训练过程中，它可以使用一种称为伪标记（pseudo-labeling）的策略，将同源 MHC 不明确的 EL 序列聚类到单个 MHC 特异性上。这不仅使发现新基序成为可能，也大幅扩大了训练集规模，从而整体提升方法的预测能力。

在本文中，我们使用 NNAlign MA 更新 NetMHCpan 和 NetMHCIIpan，增强其训练能力，并提升其预测性能。具体而言，我们将 NNAlign MA 纳入新模型的核心，从而显著扩展训练集。进一步地，我们对两个模型进行了完整的独立表位评估，并展示更新后的方法如何优于其他当前最先进的算法。

## NNAlign MA 机器学习框架

新版 NetMHCpan 和 NetMHCIIpan 与其前代方法在两个关键方面不同：训练数据和机器学习建模框架。训练数据通过汇集公共领域中的 MHC BA 与 EL 数据得到了大幅扩展。尤其是 EL 数据被扩展为包含 MA 数据。

用于训练 NetMHCpan-4.1 的组合数据集包含 13,245,212 个数据点，覆盖 250 种不同的 MHC I 类分子；用于训练 NetMHCIIpan-4.0 的组合数据集包含 4,086,230 个数据点，总共覆盖 116 种不同的 MHC II 类分子。训练集与数据划分的具体细节见补充材料。

机器学习框架从 NNAlign 更新为 NNAlign MA，以便有效处理这些 MA 数据。简而言之，NNAlign 框架是一个单等位基因框架，允许在模型训练中整合混合数据类型（BA 和 EL），从而跨不同数据类型利用信息，并提升预测能力（12,13）。NNAlign MA 扩展了该训练框架，使 EL MA 数据也可以被纳入。其实现方式是在模型训练过程中，迭代地为 MA 数据注释最佳的单等位基因，从而有效地对 MA 结合基序进行解卷积（17）。模型超参数与交叉验证训练性能的具体细节见补充材料。

## Web 界面

### 提交页面

#### 输入数据

两个服务器都接受两种不同类型的输入：FASTA 和 PEPTIDE。输入数据可以直接粘贴到提交框中，也可以从用户本地磁盘上传。对于 FASTA 输入，用户可以指定要纳入预测的肽段长度：对于 I 类，长度范围为 8-14 个氨基酸，默认值为 8-11；对于 II 类，只允许一个长度，默认值为 15。

对于 II 类，用户还可以指定是否使用 CONTEXT 编码（13）。该上下文由配体来源蛋白 N 端和 C 端区域的氨基酸组成。

提交页面包含所有可接受格式的输入数据示例，并提供按钮用于自动上传样例数据。

#### MHC 选择

接下来，服务器提供下拉菜单，用于选择要使用的 MHC 家族和分子。NetMHCpan-4.1 覆盖超过 11,000 种 MHC 分子，涵盖人类（HLA-A、HLA-B、HLA-C、HLA-E、HLA-G）、小鼠（H-2）、牛（BoLA）、灵长类（Patr、Mamu、Gogo）、猪（SLA）、马（EQCA）和犬（DLA）。NetMHCIIpan-4.0 覆盖近 1,000 种人类（HLA-DR、HLA-DQ、HLA-DP）和小鼠（H-2）MHC 等位基因。对于 DQ 和 DP，用户可以组合已覆盖的 alpha 与 beta 蛋白链。

此外，由于两种方法都具有泛特异性，只要上传 FASTA 格式的全长 MHC 蛋白序列，即可对任何已知序列的 MHC 分子进行预测。

#### 其他配置

两个 NetMHCpan 方法都会基于 %Rank 分数告知某条序列是否为强 MHC 结合肽（strong binder，SB）或弱 MHC 结合肽（weak binder，WB）。简言之，%Rank 是一种变换，用于在不同 MHC 分子之间归一化预测分数，并允许进行跨物种的 MHC 结合预测比较。

查询序列的 %Rank 值通过将其预测分数与对应 MHC 的预测分数分布相比较而得到；该分布由一组随机天然肽段估计而来。因此，某条查询序列的 %Rank 值为 1% 意味着其预测分数位于随机天然肽段预测分数的前 1%。用于检测 SB 和 WB 的 %Rank 阈值可以通过指定相应阈值进行修改：默认情况下，对于 I 类，%Rank < 0.5% 和 %Rank < 2% 分别被视为检测 SB 和 WB 的阈值；对于 II 类，%Rank < 2% 和 %Rank < 10% 分别被视为 SB 和 WB 的阈值。

此外，工具还提供一个选项，可以只报告低于某个指定 %Rank 阈值的序列；对于 II 类，如果输入格式选择了 FASTA，还可以在给定结合核心重叠范围内只打印结合最强的肽段。

另外，用户可以选择在 EL 可能性之外，同时获得输入序列的 BA 预测分数；也可以按相应的 EL 预测值从高到低对输出进行排序。为方便用户，工具还提供将输出保存为 `*.XLS` 文件的功能，该格式可被多数电子表格软件读取。

### 输出页面

两个服务器的输出都会详细列出所提供输入序列在所选 MHC 分子上的结合预测值，并给出用于解释结果的额外信息。如图 1 所示，NetMHCpan 和 NetMHCIIpan 的输出由若干纯文本列构成，这些列展示与预测结果有关的不同信息。

**图 1. NetMHCpan-4.1 和 NetMHCIIpan-4.0 工具的示例输出。**  
（A）NetMHCpan-4.1 的示例输出：输入为网络服务器提供的 FASTA 样例数据，等位基因为 HLA-A\*30:01，肽段长度为 9，其他选项保持默认。（B）NetMHCIIpan-4.0 的示例输出：输入为网络服务器提供的 FASTA 样例数据，等位基因为 DRB1\*04:34，其他选项保持默认。默认情况下，两种方法的预测分数都以 Score EL 列（某肽段成为 MHC 配体的可能性）和 `%Rank EL` 列（EL 百分位 Rank 分数）显示；如果用户选择纳入 BA 预测，相应数值也会被报告。`BindLevel` 列显示查询肽段中是否存在强结合肽（SB）或弱结合肽（WB）。`Peptide` 显示被拿来与所选 MHC 分子进行查询的肽段列表，所选 MHC 分子显示在 `MHC` 列。`Pos` 条目表示查询肽段在所选 FASTA 输入中的位置，`Core` 表示该肽段识别出的结合核心。`Identity` 是自动生成并分配给输入的 ID。其他列对应于所使用 MHC 类别相关的特定属性。关于输出中各列解释的更多细节，请参见两个网络服务器主页中的“output format”页面。

## 评估与示例

作为独立验证，本文使用 T 细胞表位数据集对模型进行了基准测试；对于 MHC I 类，还使用了 EL SA 数据。对于 MHC I 类，表位数据集取自 Jurtz 等人（12），并合并了从 IEDB 获得的一套全面的 MHC 多聚体验证表位；对于 MHC II 类，数据来自 Reynisson 等人（19）。EL SA 数据来自文献（20）。在所有情况下，数据都经过过滤，以确保与训练数据没有重叠；关于数据集的更多细节见补充材料。

对于表位数据，预测性能以 FRANK（12）衡量。也就是说，对于每一个表位-HLA 配对，使用洗脱配体可能性预测分数，对来源蛋白中的所有重叠肽段预测其与该 HLA 的结合；FRANK 值报告为预测分数高于该表位的肽段比例。使用该指标时，0 表示完美预测：已知表位在来源蛋白所有肽段中获得最高的预测结合值；0.5 则对应随机预测。

此外，本文还报告每个表位对应的 AUC，同样将来源蛋白中除该表位之外的所有重叠肽段都指定为阴性。关于 CD8 表位基准测试的更多细节，见补充表 7。对于 EL SA 数据集，本文按照补充材料“材料与方法”中“训练和测试数据”部分所述加入阴性诱饵肽段，并以 AUC、AUC0.1 和 PPV 评估性能。这里，PPV 由前 N 个预测中阳性肽段所占比例估计，其中 N 等于配体总数乘以 0.95（用于考虑潜在的 MS 污染物）。关于 EL SA 基准测试的更多信息，见补充表 8。

这些基准测试结果如图 2 所示。在这里，NetMHCpan-4.1 与 NetMHCpan-4.0（12）、MixMHCpred（18,21）、MHCFlurry（5）以及 MHCFlurry EL 进行了比较。MHCFlurry EL 是一个尚未发表的 MHCFlurry 版本，使用 EL SA 数据训练，可在 GitHub 获得（22）。在该基准测试中，由于 MixMHCpred 无法对包含 `X`（通配氨基酸符号）的肽段进行预测，这些肽段被从基准数据集中移除。NetMHCIIpan-4.0 也以类似方式与 NetMHCIIpan-3.2（23）、MixMHC2pred（11）、MHCnuggets（24）和 DeepSeqPanII（25）进行了比较。

除 NetMHCpan-4.1 与 NetMHCpan-4.0 在表位基准测试中的比较外，三个基准测试均确认：与各自基准测试中纳入的所有其他方法相比，NetMHCpan-4.1 和 NetMHCIIpan-4.0 的性能显著更优。对于 I 类表位基准测试，NetMHCpan-4.1 与 NetMHCpan-4.0 表现出相近的预测性能。与 NetMHCpan-4.0 相比，NetMHCpan-4.1 在 HLA-B 和 HLA-C 分子上，无论在表位基准测试还是配体基准测试中均显示出一致提升；这与用于训练 NetMHCpan-4.1 的 EL 数据集对这些位点覆盖范围大幅增加相一致。

还应注意，与在洗脱配体数据上评估性能时观察到的结果（19）相反，但与早期工作一致（13,19,26），当纳入上下文信息时，NetMHCIIpan-4.0 的性能出现下降（补充图 S3）。

**图 2. NetMHCpan-4.1 和 NetMHCIIpan-4.0 网络服务器的表位基准测试结果。**  
（A）CD8+ 表位基准测试的性能结果。不同方法的中位 FRANK 值为：NetMHCpan-4.1，0.00220；NetMHCpan-4.0，0.00230；MixMHCpred，0.00264；MHCFlurry，0.00383；MHCFlurry EL，0.00386。（B）CD4+ 表位基准测试的 FRANK 性能结果。不同方法的中位 FRANK 值为：NetMHCIIpan-4.0，0.0351；NetMHCIIpan-3.2，0.04825；MixMH2Cpred，0.0513；MHCnuggets，0.1219；DeepSeqPanII，0.1767。（C）MS MHC I 类洗脱配体基准测试的 PPV 性能结果。不同方法的中位 PPV 值为：NetMHCpan-4.1，0.8291；NetMHCpan-4.0，0.7940；MixMHCpred，0.7911；MHCFlurry，0.7256；MHCFlurry EL，0.7144。P 值标注为：\* P < 0.05，\*\* P < 10^-6，\*\*\* P < 10^-9。所有 P 值均使用双尾二项检验计算。箱线图中的箱体从数据的下四分位数延伸到上四分位数（第 25 到第 75 百分位），箱内线表示中位数；须线从箱体延伸，以显示非离群数据点中最极端值的范围。

## 讨论

过去几年中，大量新的 MS 洗脱 MHC 配体数据已经可用，使得对 MHC 呈递配体组进行高度丰富的表征成为可能。本文利用这些数据，并将其与 IEDB 中可用的大规模 MHC 肽结合数据相结合，开发了 NetMHCpan 和 NetMHCIIpan 工具的更新版本。这两种方法都能够预测某个肽段被 MHC I 类和 MHC II 类分子抗原呈递的可能性（以及 BA）。

两个工具均使用 NNAlign MA 机器学习框架进行训练。该框架能够整合来自表达多个 MHC 等位基因细胞系的 MS 配体数据集。与其他可用的最新算法进行基准比较后，这些方法在预测 MHC 配体和 T 细胞表位方面表现出显著提升的预测能力。

对于 NetMHCpan-4.1 和 NetMHCIIpan-4.0，性能提升在预测 MS 鉴定的 MHC 配体时最为明显。对于 I 类尤其如此；在表位基准测试中，NetMHCpan-4.1 的表现与其最近的前代 NetMHCpan-4.0 相当。对表位预测性能影响有限可能有许多原因，包括当前可用表位数据对既往预测方法和体外实验验证技术的偏倚，以及 MS EL 数据中存在、但 T 细胞表位中不共享的偏倚。未来工作将厘清这些偏倚的影响和重要性，并使我们能够评估：对 MS MHC 配体预测能力的提升，在多大程度上也能转化为 T 细胞表位预测能力的提升。

工具的基准评估表明，NNAlign MA 机器学习框架整体上具有稳健能力，能够对训练数据中包含的所有 MHC 分子进行基序解卷积。然而，结果也提示，对于由有限配体数据集表征的 MHC 分子，如 HLA-C 和 HLA-DQ，性能较低。这两个位点注释到 MHC 的配体数量较少，一部分可由其相对较低的蛋白表达水平解释；其他原因可能包括，在进行 MS 实验前纯化 MHC 分子的免疫沉淀（IP）过程中，所使用抗体的 HLA 位点特异性存在差异。未来工作可能会说明，使用具有更好 HLA-DQ 特异性的抗体，或如文献（8）所建议那样使用例如带标签 HLA 分子的工程细胞系，是否能够帮助解决这一问题。

尽管本文提出的预测方法以及其他近期发表方法性能提升的主要贡献之一，是整合了 MS 来源的 EL 数据，但 MS 数据本身也包含内在偏倚，例如会导致“可飞行”（flyable）肽段过度代表（27），并忽略含半胱氨酸的肽段（7）。这些偏倚限制了 MS 中可检测到的配体集合，因此也限制了学习到的结合基序。鉴于此，可能需要进一步发展互补性的高通量技术平台，用于检测 MHC-肽相互作用，以补全我们对 HLA 抗原呈递的理解。

NetMHCpan 和 NetMHCIIpan 都具有易于使用的用户界面，允许用户简便上传查询序列数据，并选择需要测试结合情况的 MHC 等位基因。作为目前唯一公开可用的工具，两种方法都展示出真正的泛特异性能力，使用户能够对所有 MHC 分子进行预测，包括那些此前未由结合数据表征过的分子。工具输出采用简单文本格式，并配有引导信息，帮助用户选择相关的表位或 MHC 配体候选。

鉴于其已展示出的高性能和易用性，我们预计更新后的网络服务器将成为指导未来理性表位发现项目的重要工具。

## 数据可用性

本文描述的两个网络服务器托管于：

- http://www.cbs.dtu.dk/services/NetMHCpan-4.1
- http://www.cbs.dtu.dk/services/NetMHCIIpan-4.0

这些服务器同样将通过 IEDB analysis resource 提供。

## 补充数据

补充数据可在 NAR Online 获取。

## 经费

美国国立卫生研究院 [75N93019C00001]；EIT Health 资助项目 No. [19638]。EIT Health 由 EIT 支持，EIT 是欧盟机构。开放获取出版费用由美国国立卫生研究院 [75N93019C00001] 支持。

利益冲突声明：无。

## 参考文献

1. Duan,L. and Mukherjee,E. (2016) Janeway’s Immunobiology, Ninth Edition. *Yale Journal of Biology and Medicine*, 89, 424-425.
2. Peters,B., Nielsen,M. and Sette,A. (2020) T cell epitope predictions. *Annu. Rev. Immunol.*, 38, 123-145.
3. Nielsen,M. and Andreatta,M. (2016) NetMHCpan-3.0; improved prediction of binding to MHC class I molecules integrating information from multiple receptor and peptide length datasets. *Genome Med.*, 8, 33.
4. Karosiene,E., Rasmussen,M., Blicher,T., Lund,O., Buus,S. and Nielsen,M. (2013) NetMHCIIpan-3.0, a common pan-specific MHC class II prediction method including all three human MHC class II isotypes, HLA-DR, HLA-DP and HLA-DQ. *Immunogenetics*, 65, 711-724.
5. O’Donnell,T.J., Rubinsteyn,A., Bonsack,M., Riemer,A.B., Laserson,U. and Hammerbacher,J. (2018) MHCflurry: open-source Class I MHC binding affinity prediction. *Cell Syst.*, 7, 129-132.
6. Kim,Y., Sidney,J., Pinilla,C., Sette,A. and Peters,B. (2009) Derivation of an amino acid similarity matrix for peptide: MHC binding and its application as a Bayesian prior. *BMC Bioinformatics*, 10, 394.
7. Abelin,J.G., Keskin,D.B., Sarkizova,S., Hartigan,C.R., Zhang,W., Sidney,J., Stevens,J., Lane,W., Zhang,G.L., Eisenhaure,T.M. et al. (2017) Mass spectrometry profiling of HLA-associated peptidomes in mono-allelic cells enables more accurate epitope prediction. *Immunity*, 46, 315-326.
8. Abelin,J.G., Harjanto,D., Malloy,M., Suri,P., Colson,T., Goulding,S.P., Creech,A.L., Serrano,L.R., Nasir,G., Nasrullah,Y. et al. (2019) Defining HLA-II ligand processing and binding rules with mass spectrometry enhances cancer epitope prediction. *Immunity*, 51, 766-779.
9. Bassani-Sternberg,M. and Gfeller,D. (2016) Unsupervised HLA peptidome deconvolution improves ligand prediction accuracy and predicts cooperative effects in peptide-HLA interactions. *J. Immunol.*, 197, 2492-2499.
10. Bulik-Sullivan,B., Busby,J., Palmer,C.D., Davis,M.J., Murphy,T., Clark,A., Busby,M., Duke,F., Yang,A., Young,L. et al. (2018) Deep learning using tumor HLA peptide mass spectrometry datasets improves neoantigen identification. *Nat. Biotechnol.*, 37, 55-63.
11. Racle,J., Michaux,J., Rockinger,G.A., Arnaud,M., Bobisse,S., Chong,C., Guillaume,P., Coukos,G., Harari,A., Jandus,C. et al. (2019) Robust prediction of HLA class II epitopes by deep motif deconvolution of immunopeptidomes. *Nat. Biotechnol.*, 37, 1283-1286.
12. Jurtz,V., Paul,S., Andreatta,M., Marcatili,P., Peters,B. and Nielsen,M. (2017) NetMHCpan-4.0: improved peptide-MHC class i interaction predictions integrating eluted ligand and peptide binding affinity data. *J. Immunol.*, 199, 3360-3368.
13. Barra,C., Alvarez,B., Paul,S., Sette,A., Peters,B., Andreatta,M., Buus,S. and Nielsen,M. (2018) Footprints of antigen processing boost MHC class II natural ligand predictions. *Genome Med.*, 10, 84.
14. Alvarez,B., Barra,C., Nielsen,M. and Andreatta,M. (2018) Computational tools for the identification and interpretation of sequence motifs in immunopeptidomes. *Proteomics*, 18, e1700252.
15. Garde,C., Ramarathinam,S.H., Jappe,E.C., Nielsen,M., Kringelum,J.V., Trolle,T. and Purcell,A.W. (2019) Improved peptide-MHC class II interaction prediction through integration of eluted ligand and peptide affinity data. *Immunogenetics*, 71, 445-454.
16. Nielsen,M., Connelley,T. and Ternette,N. (2018) Improved prediction of bovine leucocyte antigens (BoLA) presented ligands by use of mass-spectrometry-determined ligand and in vitro binding data. *J. Proteome Res.*, 17, 559-567.
17. Alvarez,B., Reynisson,B., Barra,C., Buus,S., Ternette,N., Connelley,T., Andreatta,M. and Nielsen,M. (2019) NNAlign MA; MHC peptidome deconvolution for accurate mhc binding motif characterization and improved t-cell epitope predictions. *Mol. Cell Proteomics*, 18, 2459-2477.
18. Bassani-Sternberg,M., Chong,C., Guillaume,P., Solleder,M., Pak,H., Gannon,P.O., Kandalaft,L.E., Coukos,G. and Gfeller,D. (2017) Deciphering HLA-I motifs across HLA peptidomes improves neo-antigen predictions and identifies allostery regulating HLA specificity. *PLoS Comput. Biol.*, 13, e1005725.
19. Reynisson,B., Barra,C., Kaabinejadian,S., Hildebrand,W.H., Peters,B. and Nielsen,M. (2020) Improved prediction of MHC II antigen presentation through integration and motif deconvolution of mass spectrometry MHC eluted ligand data. *bioRxiv* doi: https://doi.org/10.1101/799882, 19 February 2020, preprint: not peer reviewed.
20. Sarkizova,S., Klaeger,S., Le,P.M., Li,L.W., Oliveira,G., Keshishian,H., Hartigan,C.R., Zhang,W., Braun,D.A., Ligon,K.L. et al. (2020) A large peptidome dataset improves HLA class I epitope prediction across most of the human population. *Nat. Biotechnol.*, 38, 199-209.
21. Gfeller,D., Guillaume,P., Michaux,J., Pak,H.-S., Daniel,R.T., Racle,J., Coukos,G. and Bassani-Sternberg,M. (2018) The length distribution and multiple specificity of naturally presented HLA-I ligands. *J. Immunol.*, 201, 3705-3716.
22. O’Donnell,T.J., Rubinsteyn,A., Bonsack,M., Riemer,A.B., Laserson,U. and Hammerbacher,J. (2020) MHCFlurry, https://github.com/openvax/mhcflurry.
23. Jensen,K.K., Andreatta,M., Marcatili,P., Buus,S., Greenbaum,J.A., Yan,Z., Sette,A., Peters,B. and Nielsen,M. (2018) Improved methods for predicting peptide binding affinity to MHC class II molecules. *Immunology*, 154, 394-406.
24. Shao,X.M., Bhattacharya,R., Huang,J., Sivakumar,I.K.A., Tokheim,C., Zheng,L., Hirsch,D., Kaminow,B., Omdahl,A., Bonsack,M. et al. (2020) High-throughput prediction of MHC class i and ii neoantigens with MHCnuggets. *Cancer Immunol. Res.*, 8, 396-408.
25. Liu,Z., Jin,J., Cui,Y., Xiong,Z., Nasiri,A., Zhao,Y. and Hu,J. (2019) DeepSeqPanII: an interpretable recurrent neural network model with attention mechanism for peptide-HLA class II binding prediction. *bioRxiv* doi: https://doi.org/10.1101/817502, 24 October 2019, preprint: not peer reviewed.
26. Paul,S., Karosiene,E., Dhanda,S.K., Jurtz,V., Edwards,L., Nielsen,M., Sette,A. and Peters,B. (2018) Determination of a predictive cleavage motif for eluted major histocompatibility complex class II ligands. *Front. Immunol.*, 9, 1795.
27. Jarnuczak,A.F., Lee,D.C.H., Lawless,C., Holman,S.W., Eyers,C.E. and Hubbard,S.J. (2016) Analysis of intrinsic peptide detectability via integrated label-free and SRM-based absolute quantitative proteomics. *J. Proteome Res.*, 15, 2945-2959.
