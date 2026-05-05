`LVLMs`
https://arxiv.org/pdf/2603.17372
> Related：
> [[LLMs Encode Harmfulness and Refusal Separately]]
---

## MindSet

Background：安全感知失败假说 —— 图像干扰了 VLMs 对有害意图的识别能力
核心论点：这一假说有根本缺陷
1. 实验观察：VLMs 能够识别有害意图
2. 核心发现：Jailbreak 是一种独立的内部状态，而不是感知失败导致的
3. 机制解释：图像诱导的 representation shift 中存在“越狱相关分量”
4. 量化验证：检测出的 shift 能解释多种越狱现象
5. 防御应用：JRS-Rem

---

## Jailbreak as a Distinct Internal State

#### 实验设置

构造多模态数据集 $\mathcal{D}_{mm} = \mathcal{D}_{benign} \cup \mathcal{D}_{harmful}$，其中 harmful set 的文本 ==本身就显著有害==。获取三个 VLMs 的 top-layer 最后一个 token 位置的 hidden state。

#### 三条观察

![[Pasted image 20260505202605.png]]

Observation 1：三类样本（良性、拒绝、越狱）在低维投影中形成三个明显分离的聚类。==直接否定了"模型无法区分有害与无害输入"== 的说法。

![[Pasted image 20260505202925.png]]

Observation 2：
（a）计算各类样本到 “越狱质心” 的 cosine similarity，Jailbreak 样本紧密聚集在附近，benign 和 refusal 样本则远离；
（b）在每一层训练线性分类器，从 layer 2 开始 F1 Score 非常高，表明三类 hidden state 在高维空间中线性可分。

![[Pasted image 20260505203421.png]]

Observation 3：统计越狱相应中包含风险提示的比例，发现模型 “知道有害，但还是执行”

---

## Explaining VLM Jailbreaks via the Jailbreak-Related Shift

#### 概念定义

##### 图像诱导的表征偏移
给定多模态输入 $x=[I,T]$，第 $\ell$ 层的图像诱导总偏移：（多模态输入表征 - 纯文本输入表征）
$$
\Delta \mathbf{h}^{(\ell)}(x) = \mathbf{h}^{(\ell)}([I, T]) - \mathbf{h}^{(\ell)}([\emptyset, T])
$$
##### 越狱方向
定义为 Jailbreak Samples 平均表征和 Refused Samples 平均表征之差的归一化向量：
$$
\mathbf{d}^{(\ell)} = \frac{\Delta^{(\ell)}}{\|\Delta^{(\ell)}\|_2}, \quad \Delta^{(\ell)} = \boldsymbol{\mu}_{jail}^{(\ell)} - \boldsymbol{\mu}_{ref}^{(\ell)}
$$
##### 越狱相关偏移（JRS）
总偏移投影到越狱方向上的标量分量：
$$
s^{(\ell)}(x) = \Delta \mathbf{h}^{(\ell)}(x)^\top \mathbf{d}^{(\ell)}
$$
这个标量越大，说明 Vision Modal 导致表征推向越狱状态的力度越大。

#### 跨场景验证

分三个场景验证 JRS 的有效性：
1. 文本明显 harmful，图像取 MM-SafetyBench 的 SD 和 SD-TYPO
2. 文本 benign，图像 harmful，用的 SD-TYPO
3. 对抗攻击：几何变换和梯度 HADES
![[Pasted image 20260505204735.png]]
结果：Jailbreak Samples 的 JRS 在所有场景下都显著高于 Refused Samples，而 Benign Samples 的 JRS 值集中在 0 附近。

#### 用 JRS 解释 Jailbreak

##### Phenomenon 1：图像中有害视觉信息越丰富，ASR 越高
![[Pasted image 20260505205434.png]]

对 SD 图像施加不同程度的高斯噪声，结果表明 JRS 随着有效信息增加而单调增大，ASR同步上升

##### Phenomenon 2：图像与文本语义相关性越高，ASR 越高
![[Pasted image 20260505205620.png]]

用 CLIP Similarity 衡量图文相似度，分成 Sample-Level 和 Dataset-Level。结果显示相似度越大，JRS 越高，ASR 也越高。

---

## JRS-Rem

#### 算法设计
Step 1：计算每层的 JRS 标量 $s^{(\ell)}(x)$ 及其归一化值 $\tilde{s}^{(\ell)}(x) = s^{(\ell)}(x) / \|\Delta \mathbf{h}^{(\ell)}(x)\|_2$
Step 2：若归一化 JRS 超过阈值 $\tau$，则从隐藏状态中减去越狱相关分量：
$$
\hat{\mathbf{h}}^{(\ell)}(x) = \mathbf{h}^{(\ell)}(x) - s^{(\ell)}(x) \cdot \mathbf{d}^{(\ell)}, \quad \text{s.t.} \quad \tilde{s}^{(\ell)}(x) > \tau
$$
==注：只修正第一个生成 token 的前向传播，后续不变保证效率；$\tau$ 固定为 0.2==
##### 计算效率
仅需额外 2 次 token 级前向传播（一次文本一次MM），可忽略不计
##### 越狱方向的预计算
对每个 VLM，从 HADES 取 50 个越狱样本 + 50 个拒绝样本，预计算各层的 $\mathbf{d}^{(\ell)}$
