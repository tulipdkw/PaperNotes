`LVLMs` 
`Safety Attention Heads`
https://arxiv.org/pdf/2501.02029
>References:
>[[On the Role of Attention Heads in Large Language Model Safety]]
---
## MindSet

核心问题：LVLMs 比 LLMs 更容易遭受 Jailbreak。探索 LVLMs 是否知道某个输入是恶意的？这种感知能力藏在哪里？

+ 发现1：模型内部激活值在生成第一个 token 时就能区分恶意/良性输入
+ 发现2：这种能力集中在几个稀疏的 Attention Heads 中，删除他们会失守
+ 发现3：引入 Vision Modal 减少了 Safety Heads 的数量，解释了为什么 LVLMs 更脆弱
+ 应用：用这些 Heads 构建了轻量级的 Jailbreak 检测器 SAHs

---

## Findings

#### 关键抓手：Linear Probes
