`Over Refusal`
https://arxiv.org/pdf/2603.27518
---

我在四个模型上用orbench的2x2计算了

w_harm_a = JB-BA

w_harm_r = HR-OR

然后算他们的cossim,统计每个layer的情况:

Processing Llama-3.1-8B-Instruct...

Layers=33, best layer=1, cos=0.9633, final layer cos=0.3989

Processing Qwen2.5-7B-Instruct...

Layers=29, best layer=2, cos=0.7633, final layer cos=0.5136

Processing Qwen3-8B...

Layers=37, best layer=1, cos=0.9301, final layer cos=0.4174

Processing gemma-3-12b-it...

Layers=49, best layer=17, cos=0.9842, final layer cos=0.5721

发现四个模型几乎都是递减