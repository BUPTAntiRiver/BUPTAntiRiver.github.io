Formula:

$$
\text{RMSNorm}(x)= \frac{x}{\sqrt{ \frac{1}{d}\sum ^{d}_{i=1}x_{i}^{2}+\varepsilon }}\odot g
$$

- $d$: hidden dimension
- $\varepsilon$: small constant for numerical stability
- $g\in \mathbb{R}^{d}$: **learnable scale parameter**
- No bias term

Comparing to LayerNorm, RMSNorm doesn't need mean subtraction and bias term, so it is more compute efficient and it has the same good performance as LayerNorm.
