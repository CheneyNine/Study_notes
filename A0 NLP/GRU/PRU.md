# RNN存在的缺点：

1. 最后输入的单词权重占比太大（Ｕ、Ｖ都是常数）
2. 梯度消失或梯度爆炸的问题仍会存在

# Solution:
引入参数或函数体现模型对上一个状态、输入参数的适应

几个可行的猜想：
1. 改变Ｕ为函数
2. Ｕ不变，引入一个新的函数
3. 采用Hadamard product [[线性代数#Hadamard product]]，这样子就不会把不同顺序的状态混淆起来

![[Pasted image 20231212140319.png]]
Sigmoid activation makes output between 0 and 1
**PP这个函数输出数值在０～１之间，输出越接近０，表示状态更依赖当前的输入，越接近１，就越依赖过去的参数。**
# Parameters

$$
PP_t = \sigma(V_{pp}X_t + U_{pp}h_{t-1} + \beta_{pp})
$$
训练过程中输入的参数：
$$
X_t：d×1的矩阵
$$
$$
h_{t-1}：h×1的矩阵
$$
机器学习的参数：

$$V_{pp}：h×d的矩阵$$
$$U_{pp}:h×h的矩阵$$
$$\beta_{pp}:h×1的矩阵$$

![[Pasted image 20231212135206.png]]![[Pasted image 20231212141809.png]]