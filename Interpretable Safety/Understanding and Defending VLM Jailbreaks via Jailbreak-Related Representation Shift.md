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
发现 2 种纯文本输入和 3 种图文输入的隐藏状态在表示空间截然分离。说明视觉模块的引入把模型内部表示推离了 LLM zhu gan