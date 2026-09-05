# 3. Construct a Training Target

Key Terminology:

- _Conditional_: "Per single data point"
- _Marginal_: "Across the distribution of data points"

## 3.1. Conditional and Marginal Probability Path

**Dirac Distribution**: $z\in \mathbb{R}^d,\delta_{z}\quad X\sim\delta_{z}\Rightarrow X=z$

### Conditional Probability Path

$p_{t}(\cdot|z)$

1. $p_{t}$ is a distribution
2. $p_{0}(\cdot|z)=p_{\text{init}},p_{1}(\cdot|z)=\delta_{z}$

### Marginal Probability Path

$z\sim p_{\text{data}},X\sim p_{t}(\cdot|z)\Rightarrow X\sim p_{t}\ (\text{forget }z)$

1. $p_{t}(X)=\int p_{t}(X|z)p_{\text{data}}(z)\text{d}z$
2. $p_{0}=p_{\text{init}},p_{1}=p_{\text{data}}$

## 3.2. Conditional and Marginal Vector Field

### Conditional Vector Field

### Theorem (Marginalization Trick)

The marginal vector field by

$$
u^\text{target}_{t}(x)=\int u_{t}^\text{target}(x|z) \frac{p_{t}(x|z)p_{\text{data}}(z)}{p_{t}(x)}\text{d}z
$$

satisfies

$$
X_{0}\sim p_{\text{init}}, \frac{\text{d}}{\text{d}t}X_{t}=u_{t}^\text{target}(X_{t})\Rightarrow X_{t}\sim p_{t}\quad(0\leq t\leq 1)\Rightarrow X_{1}\sim p_{\text{data}}
$$

**Continuity Equation**

$$
\frac{\text{d}}{\text{d}t}p_{t}(x)= -\text{div}(p_{t}u_{t})(x)
$$

The left hand side is the _Change of probability mass at x_, the right hand side is _Outflow - inflow of probability mass from u_. $u$ is a vector field.

## 3.3. Conditional And Marginal Score Function

**Conditional Score**: $\nabla_{x}\log p_{t}(x|z)$
**Marginal Score**: $\nabla \log p_{t}(x)$
**Formula**: $$\nabla \log p_{t}(x)=\frac{\nabla p_{t}(x)}{p_{t}(x)}=\frac{\int\nabla p_{t}(x|z)p_{\text{data}}(z)\text{d}z}{p_{t}(x)}=\int \nabla \log p_{t}(x|z) \frac{p_{t}(x|z)p_{\text{data}}(z)}{p_{t}(x)}\text{d}x$$
