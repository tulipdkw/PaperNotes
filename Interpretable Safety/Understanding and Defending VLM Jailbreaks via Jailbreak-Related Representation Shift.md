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

