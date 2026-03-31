Paper link: [Gated Delta Networks: Improving Mamba2 with Delta Rule](http://arxiv.org/abs/2412.06464)

# Intro

In modern LLM, we use self-attention in Transformer architecture. The problem is self-attention scales quadratically with sequence length, leading to substantial computational demands.

To mitigate these issues, researchers explored alternatives like linear Transformer, which replace traditional softmax-based attention with kernel form dot-product-based [[Linear Attention]], substantially reducing memory requirements during inference, which also shows the essence of linear Transformer: a linear RNN with matrix-valued states.

Though simple naive linear attention performs worse than full attention, many methods and architectures are developed to improve it and achieves better and better performance. Such as incorporating data-dependent gating mechanisms akin to those in [[LSTM Explained|LSTM]], with models like GLA and [[Mamba]] as example.

In linear attention, the limitation is that the number of orthogonal key-value pairs they can store is _bounded by_ the model's dimension. When sequence length exceeds this dimension, "memory collisions" becomes inevitable, hindering exact retrieval.

Mamba2 address this limitation by introducing a simple _gated update rule_, $\mathbf{S}_{t}=\alpha_{t}\mathbf{S}_{t-1}+v_{t}k_{t}$, which uniformly decays all key-value associations at each step by a dynamic ratio, $\alpha_{t}\in(0,1)$. The problem of this method is coarse granularity. We have to update all key-value associations at the same time.

So the coming up solution is intuitive and straight forward, _gated delta rule_, it can update each key-value pair solely.
