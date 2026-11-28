Paper Link: http://arxiv.org/abs/1503.04069

LSTM is a variant of RNN, and it is unique for its gates and cell design. We can see this in its forward pass. We will talk about vanilla LSTM here.

# Forward Pass

Let $\mathbf{x}^{t}$ be the input vector at time $t$, $N$ be the number of LSTM blocks and $M$ the number of inputs.
One feature of RNN is that we have a weight for the input and a weight for the hidden state, the so called recurrent weight. So we have:

- Input weights: $\mathbf{W}_{z},\mathbf{W}_{i},\mathbf{W}_{f},\mathbf{W}_{o}\in \mathbb{R}^{N\times M}$
- Recurrent weights: $\mathbf{R}_{z},\mathbf{R}_{i},\mathbf{R}_{f},\mathbf{R}_{o}\in \mathbb{R}^{N\times N}$
- Peephole weights: $\mathbf{p}_{i},\mathbf{p}_{f},\mathbf{p}_{o}\in \mathbb{R}^{N}$
- Bias weights: $\mathbf{b}_{z},\mathbf{b}_{i},\mathbf{b}_{f},\mathbf{b}_{o}\in \mathbb{R}^{N}$

The $i,f,o$ stands for **input gate, forget gate, and output gate**. The peephole weights will be introduced later.

Then the vanilla LSTM layer forward pass can be written as:

$$
\begin{align*}
\bar{z}^t &= \mathbf{W}_z \mathbf{x}^t + \mathbf{R}_z \mathbf{y}^{t-1} + \mathbf{b}_z && \textit{block input} \\
z^t &= g(\bar{z}^t) \\
\bar{i}^t &= \mathbf{W}_i \mathbf{x}^t + \mathbf{R}_i \mathbf{y}^{t-1} + \mathbf{p}_i \odot \mathbf{c}^{t-1} + \mathbf{b}_i \\
i^t &= \sigma(\bar{i}^t) && \textit{input gate} \\
\bar{f}^t &= \mathbf{W}_f \mathbf{x}^t + \mathbf{R}_f \mathbf{y}^{t-1} + \mathbf{p}_f \odot \mathbf{c}^{t-1} + \mathbf{b}_f \\
f^t &= \sigma(\bar{f}^t) && \textit{forget gate} \\
\mathbf{c}^t &= z^t \odot i^t + \mathbf{c}^{t-1} \odot f^t && \textit{cell} \\
\bar{o}^t &= \mathbf{W}_o \mathbf{x}^t + \mathbf{R}_o \mathbf{y}^{t-1} + \mathbf{p}_o \odot \mathbf{c}^t + \mathbf{b}_o \\
o^t &= \sigma(\bar{o}^t) && \textit{output gate} \\
\mathbf{y}^t &= h(\mathbf{c}^t) \odot o^t && \textit{block output}
\end{align*}
$$

From this we can see that the cell design is deeply connected with gates. _Gates_ are how information is _controlled_ and _cells_ are how information is _stored_.

## Forget Gate

It enables the LSTM to reset its own state. This allowed learning for continual tasks such as embedded Reber grammer.

## Peephole Connections

It enables the cells to control the gates which makes precise timings easier to learn.
