# 2. Flow and Diffusion Models

## 2.1. Flow Models

Actually we are training a **vector filed model**, the flow is something that we get implicitly with the ODE.

## 2.2 Diffusion Models

Stochastic process:

- $X_{t}$ random variable $(0\leq t\leq 1)$
- $X:[0,1]\to\mathbb{R}^d,t\mapsto X_{t}$
  Vector Field:
- $u:\mathbb{R}^{d\times[0,1]}\to \mathbb{R}^d$
- Diffusion coefficient $\sigma:[0,1]\to \mathbb{R}\geq 0$
- $t\to \sigma_{t}$

## Stochastic Differential Equation (SDE)

$$
X_{0}=x_{0}(\text{initial cond})
$$

$$
\text{d}X_{t}=u_{t}(X_{t})\text{d}t+\sigma_{t}\text{d}W_{t}
$$

The first term is the change of vector field (ODE), the second term is some stochastic component (Brownian motion).

### Brownian motion

Stochastic process: $W=(W_{t})_{t=0}$

1. $W_{0}=0$
2. Gaussian increments: $W_{t}-W_{s}\sim\mathcal{N}(0,(t-s)I_{d})\ (0\leq s< t)$
3. Independent increments: $W_{t_{1}}-W_{t_{0}},\dots,W_{t_{n}}-W_{t_{n-1}}$ they are independent

### $\text{d}X_{t}$ notation

$$\frac{\text{d}}{\text{d}t}X_{t}=u_{t}(X_{t})\iff X_{t+h}=X_{t}+hu_{t}(X_{t})+hR_{t}(h)\quad\left(\lim_{ h \to 0 }R_{t}(h)=0\right)$$
$$\text{d}X_{t}=u_{t}(X_{t})\text{d}t+\sigma_{t}\text{d}W_{t}\iff X_{t+h}=X_{t}+hu_{t}(X_{t})+\sigma_{t}(W_{t+h}-W_{t})+hR_{t}(h)$$
The $R_{t}$ here is also a error term.
