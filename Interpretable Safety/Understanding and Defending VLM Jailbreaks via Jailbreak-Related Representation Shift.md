`LVLMs`
https://aclanthology.org/2025.findings-acl.186.pdf
---

## MindSet

核心问题：LLM Backbone 具有良好的安全对齐，但加上 Vision 模块之后安全性显著下降

1. 提出假设：视觉模块引入导致表示空间偏移
2. 实验验证：PCA 可视化证明多模态表示与纯文本表示存在明显分离
3. 解决方案：CMRM —— inference-time 校正表示偏移

---

## How Vision Modality Affects Model Behavior？

#### 可视化 representation shift

用 Jailbreak 数据集（有害文本 + 相关图像）构造 5 种输入变体：
1. 原始图像 + 原始文本
2. 空白图像 + 原始文本
3. 原始图像加高斯噪声 + 原始文本
4. 图像 caption + 原始文本
5. 纯文本

输入模型，取最后一个 token 在 top-layer 的 hidden state，然后进行 PCA：
![[Pasted image 20260505164934.png]]
发现 2 种纯文本输入和 3 种图文输入的隐藏状态在表示空间截然分离。说明视觉模块的引入把模型内部表示推离了 LLM Backbone 优化的分布，导致安全对齐失效。

#### 形式化建模

假设一个理想中的 VLM 表示既保留视觉信息，又不偏离 LLM 的分布，记为 $h^*(x,img)$
而当前实际中的 VLM 是 $h(x,img)$
那么可以有如下建模：$h(x,img)=h^*(x,img)+ \alpha [h(x,img')-h(x)]$
其中 $img'$ 是空白图像输入，$x$ 是纯文本输入，$\alpha$ 是偏移程度系数

如果“加入空白图像”和“纯文本输入”之间有一个统一的线性偏移方向，那么通过简单计算就可以还原出 $h*$

#### CMRM 算法

##### Step 1：提取偏移向量
1. Dataset-Level：$\mathbf{v}_{\text{data}}^l = \text{PCA} \left( \left\{ \mathbf{h}_t^{l(i)} - \mathbf{h}_c^{l(i)} \right\}_{i=1}^N \right)_{\text{第一主成分}}$
2. Sample-Level：$\mathbf{v}_{\text{sample}}^{l(i)} = \mathbf{h}_{t}^{l(i)} - \mathbf{h}_{c}^{l(i)}$
