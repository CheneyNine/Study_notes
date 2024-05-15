input has to be numbers
so we present the word in number vectors, call embedding
# One hot encoding
有多少个单词就用多长的向量
每个单词有唯一指定位置，对这个单词的这个位置就赋值1，其余为0

>[!error]
>1. the vector would be very huge
>2. similarities between words can't be established
>· 每个单词在一个维度上，彼此之间是割裂独立的

## 相似性如何度量
Cosine Similarity
$$
\text { Similarity }=\cos (\theta)=\frac{A \cdot B}{\|A\|\|B\|}=\frac{\sum_{i=1}^n A_i B_i}{\sqrt{\sum_{i=1}^n A_i^2} \sqrt{\sum_{i=1}^n B_i^2}}
$$

 详情：[[相似性度量#二、余弦距离|相似性度量的3种方法]]  

类似的，我们也可以通过这种方式来度量Words之间的相似性。我们用一个Embedding Matrix来记录所有 embedding 值
![[Pasted image 20231216221525.png]]
高度就是size of vocabulary
宽度是我们自己定义的维度
# Wishlist
wish1: low dimensional
wish2: semantically similar meaning, similar words close to each other 
## How to get such a rich word embeddings?
Idea: use language models

类比[[LINK Language Model training with Neural Networks|用神经网络训练语言模型]]，我们需要一个能够预测下一个单词的word embedding matrix 
# Word embedding training
## Backgrounds
1. text is semantic sequence
2. use nn can get a matrix to represent semantic relationship
3. 2 words similar will be mapped closely in the embedding space

## Practice: 3 steps in machine learning
### Training set
select a sequence of some words as input
use the next word as the out put label

> [!note] e.g. The dog was chased by a cat as...


#### Sliding window
每次扫动一个单词，进而构建一个dataset
构建结果如下：

|input 1|input 2|output label|
|---|---|---|
|was|chased|by|
|chased|by|a|
|by|a|cat|

但是一个方向太局限，所以选择从两个方向出发更合适

CBOW continues bags of word

#### Skip-Gram
选取一个中间目标，将周围一定的词加入，构建成数据集

|input|target|
|--|--|
|a|chased|
|a|by|
|a|red|
|a|cat|


### Model
#### 整体逻辑
1. take from one-hot encoding input
2. connected to a low dimensional hidden state of size N
3. output with the size as the input
4. map the outputs to the probabilities by using the softmax function

#### 具体解释
input: 1×M (the num of the vocabulary)

Embedding Matrix: M × N

通过线性代数的计算得到一个 1 × N 的矩阵
（可以理解就是 one hot 编码的那个 1 所在的位置在 embedding matrix 中对应的那一行），最后得到的结果我们记作一个 hidden layer

再用 hidden layer & context 计算他们之间的相似性，其中出现在 context 中的 word 会有higher probability 但是没出现的会有lower probability

最后得到一个概率

![[Pasted image 20231216231225.png]]
![[Pasted image 20231216231926.png]]

### loss function
Negative Log Likelihood

#### Naive bayes
$$
P\left(\left\{w_o\right\} \mid w_c\right)=\prod_{i \in \text { window }} \mathbb{P}\left(w_{o i} \mid w_c\right)
$$
一个 Center word的所有outer words（比如 4 个） 的 possibilities的乘积
$$
\prod_{t=1}^T \prod_{-m \leq j \leq m, j \neq 0} \mathbb{P}\left(w^{(t+j)} \mid w^t\right)
$$
对于长度为 T 的 sequence ，同时 Window size 为 m, likelihood 的计算结果如上。

We want to **maximize** this likelihood hence we will **minimize**  the negative log likelihood and use it as our loss function
$$
\mathcal{L}=-\sum_{t=1}^T \sum_{-m \leq j \leq m, j \neq 0} \log \mathbb{P}\left(w^{(t+j)} \mid w^{(t)}\right)
$$
对于不在 output label 中的词：
$$
\mathcal{L}=-\sum_{t=1}^T \sum_{-m \leq j \leq m, j \neq 0} \log \frac{\exp \left(u_o^{\top} v_c\right)}{\sum_{i \in V} \exp \left(u_i^{\top} v_c\right)}
$$
![[Pasted image 20231216234301.png]]
1. Look up embeddings
2. Calculate predictions
3. Project to outward vocabulary
##### 问题
too expensive
![[Pasted image 20231216234802.png]]
![[Pasted image 20231216234851.png]]

##### 解决办法
转换任务目标
![[Pasted image 20231216235030.png]]
![[Pasted image 20231216235049.png]]
给出一对，model 告诉我们这一对在不在 window 内
用 sigmoid 代替了 softmax
![[Pasted image 20231216235216.png]]


## Negative sampling

但目前为止，我们的数据集中所有数据都是 positive 的，得到的结果都是 1，我们需要 negative 的数据

用机器学习中的方法，我们随机加入一些不相关的数据

## Result
我们丢弃 Context 矩阵并保存 embedding 矩阵。
我们可以将嵌入矩阵用于下一个任务（可能是情感分类器）
我们可以在训练嵌入的同时完成该特定任务，以使嵌入情绪具体化
域/任务特定的嵌入和通用嵌入之间总是存在 tension relationship
这种 tension relationship 通常可以通过使用通用嵌入来解决，因为特定于任务的数据集似乎更小。



用师生模型
一个定性
一个定义
