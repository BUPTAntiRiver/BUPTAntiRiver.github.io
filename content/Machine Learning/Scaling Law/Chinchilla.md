Original paper link: http://arxiv.org/abs/2203.15556

In Chinchilla, they revisit the question: _Given a fixed FLOPs budget, how should one trade-off model size and the number of training tokens?_ To model this question, we model the final pre-training loss $L(N, D)$ as a function of the number of model parameters $N$, and the number of training tokens $D$. Since then, we change the topic into a formula:

$$
N_{\text{opt}}(C), D_{\text{opt}}(C)
= \operatorname*{arg\,min}_{\substack{N,D\text{ s.t. }\mathrm{FLOPs}(N,D)=C}}
L(N,D)
$$

How to find such optimal model parameters number and training tokens number? Obviously we can do iteration over pairs of many $(N, D)$ pairs, but we are talking about large language models, the FLOPs budget is huge, and that would be too much wasteful and inefficient. Here is the place scaling law comes in, we can take trials on smaller budgets and get some conclusion.

# Estimating the optimal parameter/training tokens allocation

We present three methods to answer the question above, in all three cases we start by training a range of models varying both model size and the number of training tokens and use the resulting training curves to fit an empirical estimator of how they should scale. The three approaches are:

- Fix model size and vary number of training tokens
- IsoFLOP profiles
- Fitting a parametric loss function

They are quite similar so I am going to only introduce IsoFLOP profiling here, since its name is the most non-straightforward one.

## IsoFLOP profiles

IsoFLOP means we fix training FLOP counts and vary model size, so the training tokens count will also vary. In the paper they set 9 different training FLOP counts ranging from $6\times 10^{18}$ to $3\times 10^{21}$ and consider the training loss for each point. This allows us to answer the proposed question directly.
![[IsoFLOP curves.png]]
We fit a parabola to each IsoFLOPs curve to directly estimate at what model size the minimum loss is achieved. Then, we fit a **power law** between FLOPs and loss-optimal model size and number of training tokens. We fit the exponents of the form $N_{\text{opt}} \propto C^{a}$ and $D_{\text{opt}}\propto C^{b}$ and they find that $a=0.49$ and $b=0.51$. Kind of same importance.

## Optimal model scaling

Though the three methods are different, they yield comparable predictions for the optimal scaling in parameters and number of training tokens with FLOPs. All three approaches suggest that as compute budget increases, model size and the amount of training data should be increased in approximately equal proportions. This is very useful empirically, we can use fewer budget to fit such curve and then predict the best model size and number of training tokens under given budget.
