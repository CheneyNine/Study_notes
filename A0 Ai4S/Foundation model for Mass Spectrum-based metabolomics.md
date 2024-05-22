![[Pasted image 20240521144318.png]]

# Encoder-DreaMS

Emergence of molecular structures from repository-scale self-supervised learning on tandem mass spectra串联质谱存储库规模自监督学习中分子结构的出现

从质谱到Molecular fingerprints分子指纹预测，并实现光谱相似性的比较以及 F 元素的存在性检测

质谱解释的现有方法：质谱相似性、正向注释和逆向注释


原则上，可以从质谱中推导出分子结构而无需依赖注释的光谱库。
为实现这一目标，我们设想三个主要的未来步骤。
首先，通过从MassIVE和其他资源库（如MetaboLights [55]）挖掘更多数据来扩展GeMS数据集。
其次，通过利用LC-MS实验中的所有可用信息，如同位素模式或在MS1光谱中的加合物分布，来提升我们的方法。
第三，将大型化合物数据库（如PubChem）纳入我们的自监督方法。受无监督翻译[56]和文本-图像对齐[57]工作启发，我们计划探索将DreaMS嵌入与小分子基础模型[58]的表示空间对齐。

from dreams.api import compute_dreams_embeddings
dreams_embeddings = compute_dreams_embeddings('data/examples/example_5_spectra.mgf')
输出结果是一个 5 行 1024 列的矩阵，对应了 5 个输入对应的 1024 维的 DreaMS representation


# QFormer 
BLIP-2，一种预训练方式，从现成的冻结*预训练图像编码器*和冻结*大型语言模型*启动视觉-语言预训练。BLIP-2通过一个***轻量级的查询转换器***，弥合了模态间的差距
The authors used a lightweight Querying Transformer to serve as a bottleneck between the frozen image and text encoders (models).
首先，图像被传递给图像编码器以提取视觉特征，然后将输出传递给语言模型以理解它。然而，存在一个挑战；由于冻结的语言模型没有在图像数据上进行训练，因此在没有进一步帮助的情况下，它无法对提取的视觉表示进行良好的解释。为了解决这个问题，Q-Former使用一组可学习的***querying vectors***，并分两个阶段进行预训练：（1）使用冻结图像编码器进行视觉-语言表示学习，（2）使用冻结文本编码器进行视觉到语言生成学习阶段。

使用一组可学习的 querying vectors 来提取相关的视觉特征，捕获与图像配对的文本中最具信息量的部分
## 训练
这种 Transformer 分两阶段进行预训练。
第一阶段从冻结的图像编码器启动视觉-语言表示学习(Encoder)
第二阶段从冻结的语言模型启动视觉到语言的生成学习(Decoder)
尽管BLIP-2的可训练参数比现有方法少很多，但在各种视觉语言任务上都达到了最先进的性能。

![[Pasted image 20240520160652.png|475]]

## Q-Former 模型架构
 由两个共享相同自注意力层的transformer子模块组成
 一个与Frozen Encoder交互用于特征提取
 另一个 Transformer 既可以作为文本 Encoder 又可以作为文本Decoder


实验设置
queries 32个，每个维度是768


![[Pasted image 20240520162550.png]]
（左）Q-Former和BLIP-2第一阶段视觉-语言表示学习目标的模型架构。我们共同优化三个目标，这些目标强制查询（即一组可学习的嵌入）提取与文本最相关的视觉表示。
（右）用于每个目标的自注意力屏蔽策略，以控制query-text交互。

## 从冻结的图像 Encoder 中Bootstrap 视觉-语言表示学习
联合优化三个输入格式和模型参数都相同的预训练目标
每个目标在query和text之间采用不同的注意力掩码策略
1. image-text 对比学习   **将图像表示与文本表示对齐，Q-Former可以捕捉与语言模型能够解释或解码的内容相一致的图像视觉特征**
2. 基于image 的 text 生成 **接受图像作为输入来使Q-Former生成文本。** 学习如何使用查询向量来提取语言模型可以理解和处理的相关视觉特征。
	基于图像的文本生成 (ITG) 损失训练 Q-Former 在给定输入图像作为条件的情况下生成文本。由于 Q-Former 的架构不允许冻结图像编码器和文本标记之间直接交互，因此必须首先通过query提取生成texts所需的信息，然后通过自注意力层传递给文本标记。因此，查询被迫提取捕获有关文本的所有信息的视觉特征。我们采用***多模态因果自注意力掩码***来控制查询文本交互，类似于 UniLM 中使用的掩码（Dong 等人，2019）。查询可以相互关注，但不能关注文本标记。每个文本标记可以处理所有查询及其先前的文本标记。我们还将 [CLS] 标记替换为新的 [DEC] 标记，作为指示解码任务的第一个文本标记。
3. image-text 匹配：学习 image 和 text representation 之间的细粒度对齐，二元分类任务，预测图像文本对之间是匹配还是不匹配。We use a ***bi-directional self-attention mask*** where all queries and texts can attend to each other


## 从 Frozen 的 LLM（Decoder）中 Bootstrap 视觉-语言生成学习
两种LLM变体：基于Decoder的 & 基于Encoder-Decoder的
图像变换器的输出查询嵌入被线性投影到与语言模型期望的文本嵌入相同的维度。然后将这些投影的查询嵌入与文本嵌入concatenate（开头），作为软视觉提示，迫使语言模型专注于Q-Former提取的视觉特征。

![[Pasted image 20240521210052.png]]

![[Pasted image 20240521211524.png]]

# Decoder-ChemGPT 
参数量：1 Billion （1G）
ChemGPT https://huggingface.co/ncfrey 1.2B/19M/4.7M
## How to use

1. Clone this repository and start editing, or save it and use it as a template for new projects.
2. Edit `lit_models/models.py` with the PyTorch code for your model of interest.
3. Edit `lit_data/data.py` to load and process your PyTorch datasets.
4. Perform interactive experiments in `prototyping.py`.
5. Scale network training to any number of GPUs using the example batch scripts.


# 代码
# Encoder
DreaMS/dreams/api.py
输出 n 个 1024 维的 DreaMS 表示
数据格式：mgf
```text
BEGIN IONS
NAME=DMAPT
DESCRIPTION=MCE bioactive compounds library
EXACTMASS=293.199094
FORMULA=C17H27NO3
INCHI=InChI=1S/C17H27NO3/c1-11-6-5-9-17(2)15(21-17)14-12(8-7-11)13(10-18(3)4)16(19)20-14/h6,12-15H,5,7-10H2,1-4H3/b11-6-/t12-,13+,14-,15+,17+/m0/s1
INCHIAUX=UJNSFDHVIBGEJZ-CMRIBGNTSA-N
SMILES=C/C1=C/CC[C@@]2(C)O[C@@H]2[C@H]2OC(=O)[C@H](CN(C)C)[C@@H]2CC1
FEATURE_ID=-1
MSLEVEL=2
RTINSECONDS=69.34
ADDUCT=[M+H]+
PEPMASS=294.20637
CHARGE=1
SPECTYPE=ALL_ENERGIES
Collision energy=60.0
FRAGMENTATION_METHOD=HCD
ISOLATION_WINDOW=1.2000000476839432
Acquisition=Crude
INSTRUMENT_TYPE=Orbitrap
SOURCE_INSTRUMENT=Orbitrap ID-X
IMS_TYPE=none
ION_SOURCE=ESI
IONMODE=Positive
PI=Tomas Pluskal
DATACOLLECTOR=Corinna Brungs
DATASET_ID=MSVPLACEHOLDERID
USI=mzspec:MSVPLACEHOLDERID:20220601_100AGC_pluskal_mce_1D1_A13_id.mzML:-1
SCANS=-1
PRECURSOR_PURITY=1.0
QUALITY_CHIMERIC=PASSED
QUALITY_EXPLAINED_INTENSITY=0.95719075
QUALITY_EXPLAINED_SIGNALS=0.91803277
Num peaks=61
42.033739 2.023
43.017743 1.244
43.041538 0.375
44.049385 0.271
46.064932 0.633
55.053921 0.247
56.049297 0.434
58.061086 0.921
58.064829 100
58.068315 4.309
58.071661 1.067
58.074808 0.3
67.053963 0.832
69.069565 0.262
79.053978 1.014
81.069616 1.317
82.064804 0.901
84.080497 0.567
91.053903 1.516
93.069577 2.084
94.064888 0.298
95.048843 0.28
95.085147 0.577
97.088348 0.201
98.059745 0.372
105.069597 1.626
106.064804 0.358
107.085253 1.159
108.080338 0.242
109.064416 0.257
109.100923 0.465
110.096075 1.043
116.070213 0.429
117.069763 0.601
119.085279 1.489
121.100884 0.483
129.069626 0.285
131.085311 0.665
133.10089 0.474
134.059753 0.937
135.117294 0.205
144.093231 0.522
145.100601 0.202
147.11673 0.298
149.13218 2.214
159.116485 2.567
161.131911 0.35
164.107071 0.243
164.143127 0.255
175.112091 0.209
177.127193 1.388
185.13264 0.432
192.174698 0.38
203.14307 0.317
222.185104 0.206
231.13818 0.57
249.148476 2.331
250.216614 0.841
251.117976 1.62
294.128296 0.191
294.206632 26.983
END IONS
```


1. **名称和描述**:

• **NAME**: DMAPT

• **DESCRIPTION**: MCE bioactive compounds library

• 这些字段标识了正在分析的化合物。

2. **精确质量和化学式**:

• **EXACTMASS**: 293.199094

• **FORMULA**: C17H27NO3

• 这些字段提供了化合物的精确分子质量和化学式，对于鉴定和表征化合物非常重要。

3. **结构信息**:

• **INCHI**和**INCHIAUX**: 国际化学标识符，这些提供了化合物结构的文本表示。

• **SMILES**: 简化分子线性输入系统，是另一种表示化合物结构的方式。

4. **质谱分析参数**:

• **MSLEVEL**: 2

• **RTINSECONDS**: 69.34

• **ADDUCT**: [M+H]+

• **PEPMASS**: 294.20637

• **CHARGE**: 1

• **SPECTYPE**: ALL_ENERGIES

• **Collision energy**: 60.0

• **FRAGMENTATION_METHOD**: HCD

• **ISOLATION_WINDOW**: 1.2

• 这些字段描述了质谱分析的具体参数，包括质谱级别、保留时间、加合物类型、前体离子质量、电荷、光谱类型、碰撞能量、碎片化方法和隔离窗口。这些参数对于理解质谱实验的条件和结果至关重要。

5. **实验和仪器信息**:

• **INSTRUMENT_TYPE**: Orbitrap

• **SOURCE_INSTRUMENT**: Orbitrap ID-X

• **ION_SOURCE**: ESI

• **IONMODE**: Positive

• **PI**: Tomas Pluskal

• **DATACOLLECTOR**: Corinna Brungs

• **DATASET_ID**: MSVPLACEHOLDERID

• 这些字段提供了实验使用的仪器类型、离子源类型、离子模式和主要研究人员的信息。

6. **质谱峰数据**:

• **Num peaks**: 61

• 列出了61个质谱峰的质量电荷比（m/z）及其对应的强度。这些峰值信息是质谱分析的核心数据，用于确定化合物的结构和性质。

# 中间层
line 536: 