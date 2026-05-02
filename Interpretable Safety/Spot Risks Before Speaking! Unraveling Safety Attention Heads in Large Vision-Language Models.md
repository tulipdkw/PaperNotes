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
![[Pasted image 20260502232410.png]]
（a）横轴：均值准确率，纵轴：方差。Top-32头集中在高准确率、低方差区域，而其他头则分散。
（b）t-SNE可视化，top-32头的激活值能将恶意和良性样本清晰分开。

#### Finding 3：Safety Heads 就是安全护盾

![[Pasted image 20260502230558.png]]

（a）重点对比随机删除 32 头和 0.1% 探针得到的 top-32 头。top-32 在通用性能（MM-Vet）不变的情况下削弱了 15% 的安全性能；而随机 32 头是整个模型都破坏了，不能说明问题。合起来说明Safety Heads 是安全功能与通用性能解耦的。
（b）从 top-4 增加到 top-64，拒绝率持续上升，但是到 32+ 之后通用性能被破坏。

#### Finding 4：引入视觉模态导致了 Safety Heads 数量的减少

![[Pasted image 20260502232616.png]]
（a）只用 MM-SafetyBench 中原始 harmful text 输入
（b）用 MM-SafetyBench 的 SD+TYPO 输入
（C）用 https://arxiv.org/abs/2402.02207 （ICML 2024）再次做 SFT 后用 SD-TYPO输入
 高准确率头和拒绝率变化趋势一致。

---

## SAHs: An Efficient andGeneralizable LVLMDefender

1. 定位 SAs：从 1-shot 开始逐步增加训练探针，重复20次取均值。选取 top-k 个准确率最高且稳定的头作为 Safety Heads $S_M^k$
2. 训练检测器：收集 SAs 在训练集上的激活值，k个头的激活向量拼接，然后训练逻辑回归二分类器 $G_M(\cdot)$
3. 在模型生成第一个 token 前向传播时同步计算判断结果，如果判定为恶意则追加 indicating prompt 来引导模型拒绝