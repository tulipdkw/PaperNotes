`LLMs` 
`Safety Attention Heads`
https://arxiv.org/abs/2410.13708

![[Pasted image 20260502162521.png]]
## MindSet

核心假设：Safety Aligned LLMs 中存在少数几个专门负责安全的 Attention Heads，如果这些头被消融，那么 LLMs 的安全护栏也会失效

+ 第一层（局部）：针对单条 harmful query，提出 Ships 指标，找出对拒绝这条 query 影响最大的 head
+ 第二层（全局）：推广到数据集层面，提出 Sahara 算法识别负责拒绝一个 harmful dataset 的 heads group
+ 第三层（机制分析）
	1. 安全头是特征提取器
	2. 预训练对安全头的形成很重要
	3. 消融安全头不影响 helpfulness

## Ships Score

提出了两种对 attention heads 进行消融（ablation）的方法：
#### UA均匀注意力：
$$
h_i^{mod} = \text{Softmax} \left( \frac{\epsilon W_q^i W_k^{iT}}{\sqrt{d_k/n}} \right) W_v^i = A W_v^i,
$$
在计算 Q\*K 时乘上一个极小的 epsilon，目的是==破坏这个头从输入序列中提取特征的能力==。附录中证明了等式的右侧部分，可化简实验。

#### SC贡献缩放：
$$
h_i^{mod} = \text{Softmax} \left( \frac{W_q^i W_k^{iT}}{\sqrt{d_k/n}} \right) \epsilon W_v^i.
$$
在乘以 W 矩阵时乘 epsilon，==线性缩小这个 head 的输出贡献==。


#### Ships Score 的计算：
$$
\text{Ships}(q_{\mathcal{H}}, \theta_{h_i^l}) = \mathbb{D}_{\text{KL}} \left( p(q_{\mathcal{H}}; \theta_{\mathcal{O}}) \parallel p(q_{\mathcal{H}}; \theta_{\mathcal{O}} \setminus \theta_{h_i^l}) \right),
$$对于一条 harmful query，以此消融 attention 部分的每个 head，计算消融前后的模型输出概率和正常情况下的 KL 散度即为 ==这个head对这条query的Ships分数==。

#### Ships 实验：
![[Pasted image 20260502164435.png]]
+ 数据集：AdvBench、JailbreakBench、Malicious Instruct
+ 模型：Llama-2-7b、Vicuna