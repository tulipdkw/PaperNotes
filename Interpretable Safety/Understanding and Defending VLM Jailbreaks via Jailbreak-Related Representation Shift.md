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

