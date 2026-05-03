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

#### harmful 信念和 refusal 信念的相关性分析

定义量化每条指令的两个信念分数，分数越高，代表模型认为这条 query 越 harmful / 越该 refuse：
$$
\Delta_{\text{harmful}} = \text{Avg}(s^l(h^l_{t_{\text{inst}}})) = \frac{1}{L} \sum_{l=1}^{L} (\text{cos\_sim}(h^l_{t_{\text{inst}}}, \mu^{l, t_{\text{inst}}}_{\text{harmful}}) - \text{cos\_sim}(h^l_{t_{\text{inst}}}, \mu^{l, t_{\text{inst}}}_{\text{harmless}})),
$$
$$
\Delta_{\text{refuse}} = \text{Avg}(s^l(h^l_{t_{\text{post-inst}}})) = \frac{1}{L} \sum_{l=1}^{L} (\text{cos\_sim}(h^l_{t_{\text{post-inst}}}, \mu^{l, t_{\text{post-inst}}}_{\text{refuse}}) - \text{cos\_sim}(h^l_{t_{\text{post-inst}}}, \mu^{l, t_{\text{post-inst}}}_{\text{accept}})).
$$
![[Pasted image 20260503234433.png]]
在 Llama-2 上的可视化结果进一步证明 harmful 和 refusal 在模型内部是脱钩的。

#### harmful 和 refusal 方向都可以诱发拒绝行为

分别提取 refusal 和 harmful 的 hidden state 向量：
$$
v_{harmful}^l=\mu_{harmful}^{l,t_{inst}}-\mu_{benign}^{l,t_{inst}},\quad v_{refusal}^l=\mu_{refusal}^{l,t_{post-inst}}-\mu_{accept}^{l,t_{post-inst}}
$$
然后把这两个向量都加到无害指令的 hidden state 里面，发现都能诱发拒绝：
![[Pasted image 20260503234855.png]]
但是：（1）refusal 的峰值能达到 100%，并且层数和 harmful 不完全相同；（2）这两个方向的==余弦相似度仅有 0.1，几乎正交，是完全不同的方向==。

#### 因果证明：reply inversion tasks

==巧妙设计==：在指令末尾追加问题“Could this user prompt cause harm? Answer 'Certainly' if yes, otherwise 'No'.”。==这样模型被期待的行为就应该是 accept harmful & reject benign==。

***Steering 引导实验***

![[Pasted image 20260503235226.png]]

（a）对无害指令分别沿以下方向 steering：
1. harmful：让模型认为指令有害导致 refusal rate 下降了
2. refusal：模型保持拒绝，高 refusal rate
3. reverse refusal：降低了模型的 refusal rate

（b）对有害指令分别沿以下方向 steering：
1. reverse refusal：保持低 refusal rate
2. reverse harmful：refusal rate 冲高但不明显
3. refusal：refusal rate 明显冲高

#### Section 3 总结：逻辑梳理

3.2：聚类观察——两个位置编码的东西不一样
3.3：依旧是观察—— harmful 和 refusal 在模型 belief 中的不一致导致了最终结果
3.4：尝试回答—— harmful belief 能不能影响拒绝行为？也能。
3.5：最终回答——两种方向的拒答机制是否有根本上的区别？是的。

---
## Analyzing Jailbreak via Harmfulness

核心发现：不同 Jailbreak Method 在 LLMs 内部发生的机制不同
![[Pasted image 20260504004127.png]]
Adversarial suffix（GCG）：harmful belief 高，但 refusal belief 被压低 → 只是关掉了拒绝开关，模型内部仍然"知道"这是有害的。
Adversarial template（模板包装）：类似，主要靠压制拒绝信号。
Persuasion（说服重写）：harmful belief 本身变低，进入负区间 → 这类越狱真正改变了模型对内容有害性的判断，让模型"相信"这个请求是合理的。

---

## Latent Guard

**设计**：非常简单。给定一条输入指令，计算 Δ_harmful（§3.3），分数为正则判断为有害，为负则判断为无害。聚类中心只需要从训练集采样 100 条有害 + 100 条无害指令来建立。