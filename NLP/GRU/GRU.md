# Leaky PRU
引入参数α来缓解梯度消失或梯度爆炸的问题，但是α作为超参数会限制网络的学习能力。

Use skip connections, aka as (也称) leaky units. Gradients can flow through the skip connection.

$$
\tilde{h}_t = \tanh(VX_t + U(PP_t \odot h_{t-1}) + \beta_1)
$$
$$
h_t = \alpha h_{t-1} + (1 - \alpha)\hat{h}_t
$$
# GRU
将超参数α转变成gate来使用
- 经过处理的ht
$$
\tilde{h}_t=\tanh \left(V X_t+U\left[R_t \odot h_{t-1}\right]+\beta_1\right)
$$
- 最终的ht
$$
h_t=Z_t \odot h_{t-1}+\left(1-Z_t\right) \odot \tilde{h}_t
$$
1. 更新门 (和[[PRU]]中的PP gate是一个概念)
$$
R_t=\sigma\left(V_R X_t+U_R h_{t-1}+\beta_R\right)
$$

2. 重置门
$$
Z_t=\sigma\left(V_Z X_t+U_Z h_{t-1}+\beta_Z\right)
$$

# 逻辑过程
1. 上一个状态ht-1和当前的输入Xt进来
2. 经过两个门的计算得到更新门和重置门的两个参数（均在[0,1]之间）
3. 更新门参数Rt和ht-1、Xt一起计算得到临时的 ht
4. 临时的 ht 和重置门参数 Zt 一起计算得到最终的 ht


![[Pasted image 20231212143041.png]]