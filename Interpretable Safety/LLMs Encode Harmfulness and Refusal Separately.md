`LLMs` 
`Activation Clustering`
`Token-level Analysis`
https://arxiv.org/pdf/2507.11878
---
![[Pasted image 20260503175934.png]]
## MindSet

核心问题：模型“认为有害”等于“会拒答”吗？
论点：在 LLMs 中，harmfulness 和 refusal 是两个独立概念，被编码在模型的不同位置。harmfulness 编码在用户指令最后一个 token 的 hidden state（$t_{inst}$）；refusal 编码在模板后缀 token 的 hidden state（例如 llama-2 中的 $t_{post-inst}$）

Contributions:
1. 通过聚类分析证明两者分离编码
2. 提取独立的“有害性”向量，用因果实验证明其真实存在
3. 用有害性表示解析不同越狱方法的内部机制
4. 提出 Latent Guard 安全检测器

---

## Decoupling Harmfulness from Refusal

