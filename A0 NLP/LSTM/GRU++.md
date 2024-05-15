# GRU存在的缺点
1. **Memory** and **output** use the same hidden state
2. Performance was limited -> 2 gates
# Idea

原先的 [[GRU#GRU|GRU]]
Candidate：
$$
\tilde{h}_t=\tanh \left(V X_t+U\left(R_t \odot h_{t-1}\right)+\beta_1\right) 
$$
Final ht:
$$
h_t=Z_t \odot h_{t-1}+\left(1-Z_t\right) \odot \tilde{h}_t

$$
改进：
Instead of resetting the previous hidden state, we can reset the candidate directly.
$$

\tilde{h}_t=\tanh \left(V X_t+U\textcolor{red}{h_{t-1}}+\beta_1\right) 
$$
$$
\tilde{h}_t=i_t \odot \tilde{h}_t
$$
用新的参数（输入门）× Candidate作为当前状态。

Memory Part: ( ft 和之前的 Zt 是一样的)
$$
c_t=\mathrm{f}_{\mathrm{t}} \odot c_{t-1}+i_t \odot \widetilde{c_t}
$$
但这样会有一个问题，ft和it加起来会超过1，导致整个结果不受限制，因此，我们需要再引入一个激活函数：
$$
h_t=\tanh \left(c_t\right)
$$
这样有两个 memory, ht是受限制的，ct是不受限制的。

在最后还加入了一个额外的门控机制ot，这使得网络更加多样化和灵活。
$$
h_t=o_t \odot \tanh \left(c_t\right)
$$
最终GRU++的变形结果：
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
# GRU++的计算逻辑
1. h(t-1)与 Xt 相结合计算出 ft/it/ot
2. h(t-1)与 Xt 相结合计算出Candidate Ct（参数还是原先的 U、V）
3. it 与 ft 结合 Candidate Ct 计算出网络中的 memory Ct
4. Ct在激活函数和 ot 的结合计算下得到 memory ht

![[Pasted image 20231213090546.png]]
 
 - 遗忘门f:  决定哪些信息被遗忘掉
 - 输入门i:  控制哪些新的信息被写入到模型中
 - 输出层o: 控制what part of cell are added into the hidden state
 - New cell content: tilde{c} 学到的新东西
 - Cell state(*Longer term*): 根据遗忘门和输入门决定忘掉多少，学到多少
 - Hidden state: 用输出门决定哪些信息被写入