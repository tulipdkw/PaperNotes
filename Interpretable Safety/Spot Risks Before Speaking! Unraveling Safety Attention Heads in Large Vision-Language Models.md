`LVLMs` 
`Safety Attention Heads`
https://arxiv.org/pdf/2501.02029
>Related:
>[[On the Role of Attention Heads in Large Language Model Safety]]
---
## MindSet

核心问题：LVLMs 比 LLMs 更容易遭受 Jailbreak。探索 LVLMs 是否知道某个输入是恶意的？这种感知能力藏在哪里？

+ Finding 1：模型内部激活值在生成第一个 token 时就能区分恶意/良性输入
+ Finding 2：这种能力集中在几个稀疏的 Attention Heads 中，删除他们会失守
+ Finding 3：引入 Vision Modal 减少了 Safety Heads 的数量，解释了为什么 LVLMs 更脆弱
+ Application：用这些 Heads 构建了轻量级的 Jailbreak 检测器 SAHs

---

## Findings

#### 关键抓手：Linear Probes
$$
g_{\theta}(a_l^h) = \text{sigmoid}(\langle \theta, a_l^h \rangle), \theta \in \mathbb{R}^D
$$
对每个 Attention Heads 单独训练一个 ==逻辑回归二分类器==（恶意/良性），用极少样本训练，衡量头对恶意输入的敏感程度。

#### Finding 1：注意力头激活值可线性分类恶意与良性输入

![[Pasted image 20260502225039.png|697]]

在 MM-SafetyBench上，对三个模型的所有注意力头，用10%数据训练线性探针。超过一半的头分类准确率超过80%，说明 ==恶意信号在模型内部线性可分==。

#### Finding 2：少数注意力头具有极强判别能力

![[Pasted image 20260502225844.png]]
减少训练样本 10% → 1% → 2 shots，大多数头准确率下降，但有少数头仍保持高准确率 + 低方差，表明他们捕捉到了最本质的安全相关表征，定义为 Safety heads。

#### Finding 3：Safety Heads 就是安全护盾

![[Pasted image 20260502230558.png]]

（a）重点对比随机删除32头和 0.1% 训练数据