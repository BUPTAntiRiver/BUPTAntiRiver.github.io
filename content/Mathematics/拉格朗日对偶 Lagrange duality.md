考虑一个存在约束条件的**原始 Primal**优化问题，有如下形式：

$$
\begin{align}
\min_w\quad &f(w)\\
\text{s.t.}\quad &g_i(w)\leq0,\quad i=1,\ldots,k\\
&h_i(w)=0,\quad i=1,\ldots,l.
\end{align}
$$

我们定义**拉格朗日表达式 Generalized Lagrangian**为：
$$\mathcal{L}(w,\alpha,\beta)=f(w)+\sum_{i=1}^k\alpha_ig_i(w) +\sum_{i=1}^l\beta_ih_i(w).$$
此处的 $\alpha_i,\beta_i$ 为**拉格朗日乘子 Lagrange mutipliers**
考虑下列等式：

$$\theta_{\mathcal{P}}(w)=\max_{\alpha,\beta:\alpha_i\geq0}\mathcal{L}(w,\alpha,\beta).$$

此处的下标 $\mathcal{P}$ 表示 "primal" 原始问题，假设选取任意违反了原始问题约束条件的 $w$ 值，那么显然：

$$
\begin{align}
\theta_{\mathcal{P}}(w)&=\max_{\alpha,\beta:\alpha_i\geq0}f(w)+\sum_{i=1}^k\alpha_ig_i(w) +\sum_{i=1}^l\beta_ih_i(w)\\
&=\infty.
\end{align}
$$

相应的，如果满足约束条件那么 $\theta_{\mathcal{P}}(w)=f(w)$。因此我们得到：

$$
\theta_{\mathcal{P}}(w)=\begin{cases}
f(w)&\text{if $w$ satisfies primal constraints}\\
\infty& \text{otherwise.}
\end{cases}
$$

如此一来我们原本的最小化问题就变成了：

$$\min_w\theta_{\mathcal{P}}(w)=\min_w\max_{\alpha,\beta:\alpha_i\geq0}\mathcal{L}(w,\alpha,\beta)$$

与原问题一致，它们有相同的解。方便后续介绍，我们设 $p^*=\min_w\theta_{\mathcal{P}}(w)$ 表示目标的最优解，我们称它为原始问题的 **值 value**。

现在，我们来看稍微有点不一样的一个问题。我们定义

$$\theta_\mathcal{D}(\alpha,\beta)=\min_w\mathcal{L}(w,\alpha,\beta).$$

这里的 $\mathcal{D}$ 表示 **对偶 Dual**。它和先前的定义不同在于我们依据 $w$ 取最小值，而非 $\alpha, \beta$。据此我们得到对偶优化问题

$$\max_{\alpha,\beta:\alpha_i\geq0}\theta_\mathcal{D}(\alpha,\beta)=\max_{\alpha,\beta:\alpha_i\geq0}\min_w\mathcal{L}(w,\alpha,\beta).$$

与原始问题相比，我们调换了取最大和最小的顺序，我们也定义对偶问题的 value 为：$d^*=\max_{\alpha,\beta:\alpha_i\geq0}\theta_\mathcal{D}(\alpha,\beta)$。对偶问题和原始问题之间存在什么联系？

$$d^*=\max_{\alpha,\beta:\alpha_i\geq0}\min_w\mathcal{L}(w,\alpha,\beta)\leq\min_w\max_{\alpha,\beta:\alpha_i\geq0}\mathcal{L}(w,\alpha,\beta)=p^*$$

当满足约束条件时，取到等号。

在先前所有假设之下，一定存在 $w^*,\alpha^*,\beta^*$ 使得 $w^*$ 时原始问题的解，$\alpha^*,\beta^*$ 是对偶问题的解。此外，它们还满足 **KKT 条件**，如下所示：

$$
\begin{align}
\dfrac{\partial}{\partial w_i}\mathcal{L}(w^*,\alpha^*,\beta^*)&=0,\quad i=1,\ldots,d\\
\dfrac{\partial}{\partial\beta_i}\mathcal{L}(w^*,\alpha^*,\beta^*)&=0,\quad i=1,\ldots,l\\
\alpha_i^*g_i(w^*)&=0,\quad i=1.\ldots,k\\
g_i(w^*)&=0,\quad i=1.\ldots,k\\
\alpha^*&\geq0,\quad i=1.\ldots,k
\end{align}
$$

在支持向量机中，他可以体现出为什么只有少数的支持向量在发挥作用，非支持向量的 $\alpha$ 都是 0。

# 为什么要进行拉格朗日对偶？

将有约束问题转化成**无约束**问题，更方便我们求解。
