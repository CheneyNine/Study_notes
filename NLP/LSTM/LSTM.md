之前[[GRU++]]的变形结果：
$$
\begin{aligned}
\tilde{c}_t & =\tanh \left(V X_t+U h_{t-1}+\beta_1\right) \\
c_t & =\mathrm{f}_{\mathrm{t}} \odot c_{t-1}+i_t \odot \widetilde{c_t} \\
h_t & =\mathrm{o}_{\mathrm{t}} \odot \tanh \left(c_t\right) \\
\mathrm{f}_{\mathrm{t}} & =\sigma\left(V_f X_t+U_f h_{t-1}+\beta_f\right) \\
\mathrm{i}_{\mathrm{t}} & =\sigma\left(V_i X_t+U_i h_{t-1}+\beta_i\right) \\
\mathrm{o}_{\mathrm{t}} & =\sigma\left(V_o X_t+U_o h_{t-1}+\beta_o\right)
\end{aligned}
$$

 
# LSTM 的引入
1. 1995 年引入以解决梯度消失的问题*！改善了问题而不是完全解决*
2. 训练的时候结合使用实时循环学习和随时间反向传播进行
3. 进行了多项重大修改->包括遗忘门、窥视连接等
# LSTM 的变体
![[Pasted image 20231213091357.png]]

# LSTM的缺点
1. 不能完全消除梯度消失/梯度爆炸的问题
2. Position Bias的问题也会存在

# 未来发展
第三个通道？
dimensions
size and performance
