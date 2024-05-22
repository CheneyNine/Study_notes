# Word2Vec

The algorithm was built on the idea of the **distributional hypothesis**.(分布假设)

>[!note] Distributional Hypothesis
>The distributional hypothesis suggests that words *occurring in similar linguistic contexts will also have similar semantic meaning*. Word2Vec uses this concept to map words having *similar semantic meaning geometrically close to each other* in a N-Dimensional vector space.

Word2Vec 是词嵌入领域的重大突破，因为它能够捕获以前从未捕获过的代数表示中的关系。
## Two techniques
### Common Bag of Words(CBOW) 

### Skip Gram
我们向模型提供一个单词，模型尝试预测它周围的单词。


# Glove

Word2Vec technique relies on **local statistics** (local context surrounding the word)

Glove technique goes one step further by combining local statistics with **global statistics** such as **Latent Semantic Analysis**(Word Co-occurrence Matrix): 兼顾“本地”与“全局”

From the ratio of probabilities, we can observe that **non-discriminative words**(_water & fashion)_ have *a ratio of probabilities approximately equal to 1*, whereas **discriminative words**(_solid and gas)_ have either *high* ratio of probabilities or *very low* ratio of probabilities. 

Using this method, the ratio of probabilities encodes *some crude form of meaning associated with the abstract concept of thermodynamic phases*( 热力学抽象概念的某种粗略意义 ). These ratios of probabilities are then encoded in the form of vector differences in N-Dimensional space.


# ELMo: embeddings from language models

Glove and Word2Vec will produce the same vector for the word *date* in both the sentences. Hence, our models would fail to distinguish between the **polysemous** (having different meaning and senses).

**ELMo** resolves this issue by *taking in the whole sentence as an input* rather than a particular word and generating *unique ELMo vectors for the same word* used in different contextual sentences.

ELMo uses Bi-directional LSTM, which is pre-trained on a large text corpus to produce word vectors. 此外还会学习特定目标权重，针对不同task、不同的 layer 有不同的 weight


![[Pasted image 20231220222603.png]]

不同的 Layer 有不同的作用：

![[Pasted image 20231220223226.png]]

## Representation:
### _Contextual_:
The ELMo vectors produced for a word depends on the context of the sentence the word is being used in.

### _Character based_: 
ELMo representations are purely character based, allowing the network to use morphological clues to form robust representations for out-of-vocabulary tokens unseen in training.(ELMo 表示完全基于字符，允许网络使用形态线索为训练中未见的词汇外标记形成稳健的表示。)


# Applications
1. 改进文档搜索和信息检索
calculate **Word Centroid Similarity** 计算单词质心相似度

2. improve Language Translation System
Facebook 最近发布了多语言词嵌入 (fastText),被认为是最高效的 SOTA 基线之一

3. improve Text Classification accuracy
提高了情感分析、垃圾邮件检测和文档分类等不同领域的文本分类准确性。