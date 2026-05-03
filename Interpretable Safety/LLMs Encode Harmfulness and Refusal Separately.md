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

#### Observation：去除后缀 token 会削弱拒绝能力

把 \[/INST\] 等模板后缀去除，拒绝率大幅下降。说明拒绝行为对后缀 token 存在强烈依赖。
由此产生核心假设。

#### 聚类分析：两个位置编码不同特征

Step 1：收集 4 类指令在两个位置的 hidden state
1. refused + harmful
2. accepted + harmful
3. accepted + benign
4. refused + benign
Step 2：以 refused harmful 和 accepted benign 的 hidden state 的均值，作为两个聚类中心；然后计算每条指令和两个中心的余弦相似度：
$$
s^l(h^l) = \text{cos\_sim}(h^l, \mu^l_{\text{refused harmful}}) - \text{cos\_sim}(h^l, \mu^l_{\text{accepted harmless}})
$$
Step 3：观察 refused benign 和 accepted harmful 落在哪个聚类
![[Pasted image 20260503231807.png]]
+ 在 $t_{inst}$，refused benign 更靠近 accepted benign，accepted harmful 更靠近 refused harmful。说明 $t_{inst}$ 这个位置的 hidden state 可以用来区分 benign 和 harmful；
+ 在 $t_{post-inst}$，refused benign 更靠近 refused harmful，accepted harmful 更靠近 accepted benign。说明 $t_{post-inst}$ 这个位置的 hidden state 导致了 refuse 还是 accept。

#### 提取方向

#### 因果证明：reply inversion tasks

==巧妙设计==：在指令末尾追加问题“Could this user prompt cause harm? Answer 'Certainly' if yes, otherwise 'No'.”。==这样模型被期待的行为就应该是 accept harmful & reject benign==。

