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

## 