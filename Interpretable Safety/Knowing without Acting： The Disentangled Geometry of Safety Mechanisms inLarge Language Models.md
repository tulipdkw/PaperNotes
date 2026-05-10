`LLMs`
`Axis Extraction`
`Layer-Level Analysis`
https://arxiv.org/pdf/2603.05773
> Related:
> [[LLMs Encode Harmfulness and Refusal Separately]]
> [[On the Role of Attention Heads in Large Language Model Safety]]
---

## MindSet

核心问题：Aligned LLMs 明知道请求是有害的，为什么还是会 fail to refuse？这说明 “harmfulness recognization” 和 “refuse” 之间存在脱耦

核心假说：Disentangled Safety Hypothesis（RSH）—— 模型的安全计算并非一个整体，而是分解在两个独立的子空间上

|         轴          |       符号       |       功能        |
| :----------------: | :------------: | :-------------: |
| Recognization Axis | $\mathbf{v}_H$ | 识别有害请求（knowing） |
|   Execution Axis   | $\mathbf{v}_R$ | 触发拒绝动作（Acting）  |
两轴在 early layers ==拮抗耦合== （cos_sim=-0.9）；在深层几何解耦。这条演化轨迹被命名为 **"Reflex-to-Dissociation"**。

Contributions：
1. 提出 DSH 假说
2. 绘制 Reflex-to-Dissociation 轨迹
3. 提出方法论工具：
	1. Double-Difference Extraction：从激活空间中精确分离 $\mathbf{v}_H$ 和 $\mathbf{v}_R$
	2. Adaptive Casual Steering：自适应干预这两个轴来防御
4. 发现模型架构分歧：
	- Llama3.1："显式语义控制"，拒绝向量直接映射到词汇空间的法律/道德词汇
	- Qwen2.5："隐式分布控制"，安全机制分布在无法线性映射到词汇的潜在子空间中，因此更难被攻击

---

## Method

#### 残差流的线性分解

假设 Hidden State 由四个语义分量线性叠加：
$\mathbf{h} = \mathbf{v}_{base} + \mathbf{v}_{harm} + \mathbf{v}_{refusal} + \mathbf{v}_{art}$
（基础语言能力、有害语义编码、拒绝行为驱动、结构性噪声）

定义两种状态：
+ Canonical（正常）
+ Masked：用 [[On the Role of Attention Heads in Large Language Model Safety]] 中的 Sahara 算法识别并消融安全头，拒绝机制关闭，$\mathbf{v}_{refusal} 和 \mathbf{v}_{art}$ 清除。`为什么安全头清除refusal，难道没有可能也清除harmful吗？`




