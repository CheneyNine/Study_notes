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


# Decoder-ChemGPT 
参数量：1 Billion （1G）
ChemGPT https://huggingface.co/ncfrey 1.2B/19M/4.7M
## How to use

1. Clone this repository and start editing, or save it and use it as a template for new projects.
2. Edit `lit_models/models.py` with the PyTorch code for your model of interest.
3. Edit `lit_data/data.py` to load and process your PyTorch datasets.
4. Perform interactive experiments in `prototyping.py`.
5. Scale network training to any number of GPUs using the example batch scripts.


