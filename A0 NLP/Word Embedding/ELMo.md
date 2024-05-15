 # Recap: Language Models

语言模型就是用之前的单词来预测下一个单词
 ![[Pasted image 20231229142448.png]]

## 结构
4 个部分：输入处理、embedding、特征提取、任务执行
### Input preprocessing

From corpus -> vocabulary
remove Upper letter/emoji/punctuation/grammatical mistakes


### Embedding

This stage transforms the data to be fed to the network. The embedding layer transforms every word into *a fixed length* vector with arbitrary dimension. 嵌入层将每个单词转换为任意维度的固定长度向量。 One-hot-encoding or word2vec are some examples.

### Feature Extraction

Text->vector-> another vector
another vector stands for a feature

From the sequence of embeddings, we generate a new representation which will be used in the next stage. Its job is to capture as much contextual information as possible.
根据 embedding 序列，生成新的表示向量，获得尽可能多的上下文信息。

represent depend on the context
### Task

Example：Predict
在训练过程中，使用交叉熵损失将归一化预测与真实值 y 比较（y一般是 one hot 编码的）
训练完模型后就将 Neural Network 固定并不改变
![[Pasted image 20231229145724.png]]

# Recap: Neural Networks as Language Models

![[Pasted image 20231229151236.png]]
# ELMo
## Bidirectional two-layer LSTM Language Model
To create the ELMo embeddings we could use an LSTM network.

### Two layer
采用双层结构来提高模型的性能
![[Pasted image 20231229151410.png]]

为了解决优化问题，我们采用 [[4. 深度学习#解决梯度消失的问题]]  中的残差网络 residual connection
![[Pasted image 20231229151847.png]]
其中 下一个状态 h会用第一个 LSTM 模型的输出结果结合上原有的 X_t来进行运算，其中 W 表示一个线性变换操作，用于调整 Xt 的输入
![[Pasted image 20231229152052.png]]
###  bidirectional 双向
之所以采用双向的 LSTM： 是因为语义信息不仅仅是由过去决定的，未来的信息也很重要。
The softmax layers for the forward and the backward LSTM are the same. But W is different.
![[Pasted image 20231229152725.png]]
We optimize both networks at the same time. For each word, we optimize both RNNs jointly:

$$
L = -\sum y \log(\hat{y}_{\text{right}}) - \sum y \log(\hat{y}_{\text{left}})
$$

## Character CNN  
### 原先的问题
会出现 Unkown，且没有关注 context
因此采用 characters 会很方便
### 具体操作
用三个形状不同的卷积核来提取特征
一个形状的卷积核做n次处理，每次卷积核中的具体数字也不一样
![[Pasted image 20231229162136.png]]
为了让每个单词形成的长度都是一样的，我们采用池化操作
![[Pasted image 20231229162311.png]]
这样子，我们得到了一个能够从 Words 中提取 information 并且输出长度一致的 embedding
但这些信息仅与 CNN 提取到的局部特征相关。我们需要以某种方式结合这些信息。
#### Highway Networks

>[!quate] ChatGPT
>Highway Networks 是一种神经网络结构，它引入了一种称为“门控”（gating mechanism）的概念，允许信息无障碍地穿过网络中的多个层。这种结构是由 Srivastava 等人在2015年提出的，目的是解决在训练深层网络时常见的问题，如梯度消失和表示瓶颈。
>
在传统的神经网络中，每一层的输出是前一层的输出经过加权、偏置并应用激活函数后的结果。随着层数的增加，信息和梯度必须通过所有这些变换，这可能导致训练困难，尤其是在梯度基于反向传播算法时。
>
为了克服这些问题，Highway Networks 引入了两个新的操作：转换门（transform gate）和载入门（carry gate）。这些门控制着信息是应该被传递到下一层，还是应该在当前层进行变换。
>
>1. **转换门（T）**：它决定了多少原始信息要经过变换。这个门的输出是0到1之间的值，与当前层的输入相乘，这个乘积决定了多少输入信息会被变换。
>
>2. **载入门（C）**：这个门与转换门相对应，它决定了多少信息不经变换直接传递到下一层。载入门通常被设置为 \(1 - T\)，这样就保证了一部分信息总是能够通过，即使是在深层网络中。
>
通过这种结构，Highway Networks 允许信息在必要时经过多个层的复杂变换，同时也允许信息直接通过，这样有助于保持梯度流和信息流，从而缓解梯度消失的问题，提高了网络的训练效率。
>
总结一下，Highway Networks 通过引入门控机制，提供了一种灵活的方式来平衡信息在网络中的变换和直接传递，使得深层网络的训练变得更加可行。

![[Pasted image 20231229163355.png]]
 ![[Pasted image 20231229170820.png]]![[Pasted image 20231229163417.png]]
  C=1-T
  CNN 的输出是每个单词的字符级特征表示。这些表示随后通过一个或多个 Highway Networks 层，Highway 层的作用是选择性地传递和转换信息。在 ELMo 中，Highway Networks 允许某些信息（比如字符级的特征）不经变化地直接传递，同时也可以对信息进行一定的转换。
## ELMo embeddings 
elmo vector: 对于每个单词都有一个 ELMo vector，其中有五个向量，
![[Pasted image 20231229171031.png]]
为了便于表示，用下面三个符号代替
$$
h_t^1 = \{h_t^{1l}, h_t^{1r}\} 
$$
$$

h_t^2 = \{h_t^{2l}, h_t^{2r}\}

$$
$$
h_t^0 = x_t
$$

### ELMo embeddings training
![[Pasted image 20231229171833.png]]
下面这些都要训练
![[Pasted image 20231229172001.png]]

## how we use them

We use the embeddings as our sauce. 
将 ELMo embeddings 与我们想要的embedding 目标连接起来

![[Pasted image 20231229174616.png]]
### Secret Sauce
 ELMo embedding give us the flexibility to fine-tune and adjust the embeddings *for our specific needs*.
![[Pasted image 20231229180100.png]]

The scalar 𝜸 and the vector 𝒔 are trained by the model. The value of 𝒔  is softmax normalized.

𝜸 和 𝒔 都可以根据我们的需要来进行设置调整，比如对于低字符级嵌入收益更高的模型，可以设置[1,0,0]的模型，具体如下：
![[Pasted image 20231229180314.png]]
当然也可以设置不同的比例，来做更加精确的控制