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
#### UA：无差别注意力
$$
h_i^{mod} = \text{Softmax} \left( \frac{\epsilon W_q^i W_k^{iT}}{\sqrt{d_k/n}} \right) W_v^i = A W_v^i,
$$
#### SC：贡献缩放