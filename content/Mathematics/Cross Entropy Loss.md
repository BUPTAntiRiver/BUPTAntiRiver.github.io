Usually used in discrete target distributions like classification problems.

$$
H(p,q)=-\sum_{i}p_{i}\log q_{i}
$$

If $p$ is the real distribution, then it is one-hot, so it collapse to:

$$
\text{CE}=-\log q_{\text{true class}}
$$

This is the standard case.
