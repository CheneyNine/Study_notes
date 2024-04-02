# 为什么需要注意力机制
## 语言模型 Wishlist
- 在词语之间有很好的上下文联系
- 词语能够记录序列信息
- 提高训练速度->能够平行训练的一个模型
# 注意力机制基础
1. 用位置关系来代表注意力
但这个效果不是很好，因为这个上下文和注意力的分布是不均匀的。
![[Pasted image 20240112192803.png|500]]

2. 用一个变换矩阵为不同的位置赋予不同的权重
![[Pasted image 20240112192831.png|450]]

句子输入进模型后，先变成向量，在相互计算他们的余弦相似度

![[Pasted image 20240112194454.png|300]]
用一个矩阵表示就是：
![[Pasted image 20240112194744.png|500]]
先用两个 v 计算相似度，再用 v×相似度α 得到最终结果 z(计算α前要归一化)
![[Pasted image 20240112194916.png|575]]

![[Pasted image 20240112194941.png|575]]

结构总览：
![[Pasted image 20240112195006.png|525]]




## 空间注意力模型的问题

- No weights are trained in the process
- Attention as defined to be cosine similarity leads to fixed contextual mapping: two words will always have the *same cosine similarity* irrespective of the context
- There is no positional information

## 类比改进

来比数据库：咨询+钥匙=结果
![[Pasted image 20240112195203.png|475]]
我们可以根据此设计三个矩阵来分别训练不同的权重，这样子这个变换过程就能够根据任务灵活调整
![[Pasted image 20240112195341.png|450]]
结构总览：
![[Pasted image 20240112195544.png|525]]
这就是自注意力机制的基本组成

## 问题
- Attention leads to limited contextual mapping.
    
- There is no positional information encoded.
# 建立 Transformer
## 多注意力块
We could stack up these self-attention blocks to get richer and more complex contextual relationships
![[Pasted image 20240112195651.png|400]]
![[Pasted image 20240112195900.png|500]]
我们在叠加多个注意力块的同时，也要注意梯度流动的问题，加入一些跳跃结构？添加归一化层来解决梯度问题，最后结果就是Transformer的一层 layer ⬇️
![[Pasted image 20240112200014.png|475]]
在此之外我们可以引入多层 Transformer
## 位置编码


## 组合