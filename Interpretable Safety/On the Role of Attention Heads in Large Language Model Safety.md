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
+ 模型：Llama-2-7b、Vicuna-7b-v1.5
+ 横轴：Vanilla 正常输入，Greedy 消融后贪心解码，Top-5 消融后 Top-5 解码
Ablation Head 选取：把 dataset 里面每一个 query 都测一次 Ships Score，取得最高分次数最多的 head 作为这个 dataset 对应的 safety head 被消融。
效果：用 UA 在 llama2 上的效果最明显， 消融后每个 setting 下的 ASR 都显著升高。


## Sahara Algorithm

#### 泛化的 Ships Score（数据集层）：
$$
\mathrm{Ships}(Q_{\mathcal{H}}, h_i^l) = \sum_{r=1}^{r_{main}} \phi_r = \sum_{r=1}^{r_{main}} \cos^{-1} \left( \sigma_r(U_{\theta}^{(r)}, U_{\mathcal{A}}^{(r)}) \right),
$$
对于 Harmful Dataset，取出每个样本输入到 LLMs 中后顶层的残差激活 a（位置：输入最后一个token，输出开始之前），堆叠成矩阵 M，对齐应用 SVD 分解 $\text{SVD}(M)=U \Sigma V^T$。计算每个 Attention Head 消融之后和正常情况下的 U 矩阵前 r 个主成分的向量夹脚之和，作为Ships Score。

#### Sahara 算法：
贪心策略，迭代 S 次