There are two main information of embedding positional information in transformer: absolute position and relative position.

# Background Works

An example of **absolute position** is the original work of attention is all you need.

The **relative** case is they add a trainable relative positional embedding to both $f_{k}$ and $f_{v}$, which names $\tilde{p}_{r}^{k}$ and $\tilde{p}_{r}^{v}$ respectively, where $r=\text{clip}(m-n,r_{\min},r_{\max})$ represents the relative distance between position $m$ and $n$. They clipped the distance with the assumption that precise relative information is not useful beyond a certain distance.

# Proposed Approach

## Formulation

The ultimate goal of relative information embedding is to find out a function such that:

$$
\langle f_q(x_m, m), f_k(x_n, n) \rangle = g(x_m, x_n, m - n)
$$

The inner product of positional embedded $q,k$ can be represented by a function of input and their distance.

## Rotary position embedding

We begin with a simple 2-dimension case. They find a form satisfy the equation which is:

$$
\begin{align}
f_q(x_m, m) &= (W_q x_m)e^{im\theta} \\
f_k(x_n, n) &= (W_k x_n)e^{in\theta} \\
g(x_m, x_n, m-n) &= \mathrm{Re}\!\left[(W_q x_m)(W_k x_n)^{*} e^{i(m-n)\theta}\right]
\end{align}
$$

The inspiration comes from 2-dimension vector relation.

If we push the case to $n$ dimension, we will get:

$$
f_{\{q,k\}}(x_{m},m)=R^{d}_{\Theta,m}W_{\{q,k\}}x_{m}
$$

where

$$
R^{d}_{\Theta,m} =
\begin{pmatrix}
\cos m\theta_{1} & -\sin m\theta_{1} & 0 & 0 & \cdots & 0 & 0 \\
\sin m\theta_{1} & \cos m\theta_{1} & 0 & 0 & \cdots & 0 & 0 \\
0 & 0 & \cos m\theta_{2} & -\sin m\theta_{2} & \cdots & 0 & 0 \\
0 & 0 & \sin m\theta_{2} & \cos m\theta_{2} & \cdots & 0 & 0 \\
\vdots & \vdots & \vdots & \vdots & \ddots & \vdots & \vdots \\
0 & 0 & 0 & 0 & \cdots & \cos m\theta_{d/2} & -\sin m\theta_{d/2} \\
0 & 0 & 0 & 0 & \cdots & \sin m\theta_{d/2} & \cos m\theta_{d/2}
\end{pmatrix}
$$

The big matrix above is of shape $d\times d$ and the $m$ in it is the position of the query or key to be processed, the query and key has shape $N\times d$, $N$ for sequence length, so we are applying a pair rotation to query and key, this is the essence of RoPE. Finding out such method is really impressive.

Base on their evaluation, RoPE also can show the decay of $S$ with the relative distance $m-n$ increases by setting $\theta_{i}=10000^{-2i/d}$.

If you really do the matrix multiplication you will get something looks like sine and cosine of $\theta_{i}-\theta_{j}$, which is exactly how we measure rotation of vectors.

# Proportional RoPE

The model context length is becoming larger and larger, we have context length like 128 k, so the $m$ in the example above will be more than 10000, which means we might use up the whole round of rotation. In this case we need proportional RoPE that scales down the constant in $\theta$ to ensure a valid rotation transformation. Usually we do something like dividing with $L$ (context length) to compress the rotation range.

Imagine the original RoPE transformation angle according to position $p$ is:

$$
\theta_{p}=p\cdot \omega
$$

Now it becomes:

$$
\theta_{p}=f(p)\cdot \omega=\frac{p}{L}\cdot \omega
$$

There are other scaling methods too, like linear scaling, logarithmic scaling or adaptive scaling. I will check this topic in more detail in the future.
