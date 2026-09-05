Reaches linear scaling in sequence length and fast inference, also achieves state-of-the-art performance.

# Introduction

The efficacy of self-attention is attributed to its ability to route information densely within a context window, allowing it to model complex data. However, this property brings fundamental drawbacks: an inability to model anything outside of a finite window, and quadratic scaling with respect to the window length.

Recently (well at that time), structured state space sequence models (SSM) have emerged as a promising class of architectures for sequence modeling. It can be interpreted as a **combination** of recurrent neural networks (**RNN**) and convolution neural networks (**CNN**).

In previous works, they have good performance in domains involving **continuous** signal data like **audio** and **vision**, but they don't have such one on modeling **discrete** and information-dense data such as **text**.

They proposed **selective state space models**, a new class of methods, which has fast speed and Transformer like performance.

## Contributions

**Selection Mechanism**. Enable the model to _select_ data in an input-dependent manner.

**Hardware-aware Algorithm**. Does not materialize the expanded state in order to avoid IO access (like FA). It computes the model recurrently with a scan instead of convolution. Reaches real linear rather than pseudo-linear for convolution-based SSMs.

**Architecture**. Mamba Out!

# State Space Models

Structured state space sequence models (S4) are a class of sequence models for deep learning that are broadly related to RNNs, and CNNs, and classical state space models.

S4 models are defined with four parameters $(\Delta,A,B,C)$, which define a sequence-to-sequence transformation in two stages.

## Discretization

Transform the "continuous parameters" $(\Delta, A,B)$ to "discrete parameters" $(\bar{A}, \bar{B})$ through fixed formulas $\bar{A}=f_{A}(\Delta,A)$ and $\bar{B}=f_{B}(\Delta ,A,B)$, where the pair $(f_{A},f_{B})$ is called a _discretization rule_.

## Computation

After the parameters have been transformed from $(\Delta,A,B,C)\mapsto(\bar{A},\bar{B},C)$, the model can be computed in two ways, either as **linear recurrence** or **global convolution**.

# Selective State Space Models

## Motivation: Selection as a Means of Compression

They argue that a fundamental problem of sequence modeling is _compressing context into a smaller state_. For example, attention is effective but inefficient because it explicitly does not compress context at all. It needs to store the entire context (KV cache), cause it to have slow linear-time inference and quadratic-time training. While, recurrent models are efficient because they have a finite state, implying constant-time inference and linear-time training.

The constant dynamics of $\bar{A},\bar{B}$ cannot let Linear Time Invariance (LTI) models select the correct information from their context, or affect the hidden state passed along the sequence in an input-dependent way. So they propose that a fundamental principle for building sequence models is **selectivity**.

## Improve SSMs with Selection

Let model's parameters that affect interactions along the sequence be input-dependent.

The main difference is simply making several parameters $\Delta,B,C$ functions of the input, along with associated changes to tensor shapes throughout. They add a length dimension $L$ to change the model from time-invariant to time-varying now.

## Efficient Implementation of Selective SSMs

## Properties of Selection Mechanisms

The selection mechanism is a broader concept that can be applied in different ways, such as to more traditional RNNs or CNNs, to different parameters, or using different transformations $s(x)$.

### Connection to Gating Mechanisms

The classical gating mechanism of RNNs is an instance of our selection mechanism for SSMs. RNN gating is just like the discretization of continuous-time systems.

### Interpretation of Selection Mechanisms

We elaborate on three particular mechanistic effects of selection.

#### Variable Spacing

Selectivity allows filtering out irrelevant noise tokens that may occur between inputs of interest.

#### Filtering Context

Selectivity enables model to process longer context by filtering irrelevant context.

#### Boundary Resetting

When processing a long sequence consist of multiple independent sentences, Transformer can use masking to separate them, while traditional LTI cannot do so, however with selectivity, we can reset states at boundaries to achieve this.

#### Interpretation of $A$

$A$ could also be selective, but the author think $\Delta$ can have enough selectivity in $(\bar{A},\bar{B})$, and this is the main source of improvement.

#### Interpretation of $B$ and $C$

In an SSM, modifying $B$ and $C$ to be selective allows finer-grained control over whether to let a input $x_{t}$ into the state $h_{t}$, or the state into the output $y_{t}$.
