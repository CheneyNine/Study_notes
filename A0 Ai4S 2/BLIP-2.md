BLIP-2，一种预训练方式，从现成的冻结*预训练图像编码器*和冻结*大型语言模型*启动视觉-语言预训练。BLIP-2通过一个***轻量级的查询转换器***，弥合了模态间的差距
## 训练
这种转换器分两阶段进行预训练。
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
1. image-text 对比学习
2. 基于image 的 text 生成
	基于图像的文本生成 (ITG) 损失训练 Q-Former 在给定输入图像作为条件的情况下生成文本。由于 Q-Former 的架构不允许冻结图像编码器和文本标记之间直接交互，因此必须首先通过query提取生成texts所需的信息，然后通过自注意力层传递给文本标记。因此，查询被迫提取捕获有关文本的所有信息的视觉特征。我们采用***多模态因果自注意力掩码***来控制查询文本交互，类似于 UniLM 中使用的掩码（Dong 等人，2019）。查询可以相互关注，但不能关注文本标记。每个文本标记可以处理所有查询及其先前的文本标记。我们还将 [CLS] 标记替换为新的 [DEC] 标记，作为指示解码任务的第一个文本标记。
3. image-text 匹配：学习 image 和 text representation 之间的细粒度对齐，二元分类任务，预测图像文本对之间是匹配还是不匹配。We use a ***bi-directional self-attention mask*** where all queries and texts can attend to each other


## 从 Frozen 的 LLM（Decoder）中 Bootstrap 视觉-语言生成学习

