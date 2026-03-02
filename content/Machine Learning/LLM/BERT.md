Paper link: http://arxiv.org/abs/1810.04805

Though BERT is kind of an old ancient gadget, but not every scenario needs super large language models, and BERT is still being widely used in many cases! Let's dive into it and review its success.

# Introduction

In general, self attention in Transformer and accept all information in the sequence, but for generative task, especially question answering, we usually mask out the future tokens to mimic the process. But BERT says such restriction is harmful for fine-tuning tasks at sentence-level but not token-level. So they improved fine-tuning based approaches by proposing BERT: **Bidirectional Encoder Representations for Transformer**.

It uses a masked language model pre-training objective to alleviates the previously mentioned constraint, which means some tokens of input will be randomly masked, and the model should try to predict the original tokens based only on its context.

Besides masked language model, they also use a "next sentence prediction task" that jointly pre-trains text-pair representations.

# BERT

There are two steps in the framework of BERT: _pre-training_ and _fine-tuning_. They first pre-train the model with unlabeled data over different pre-training tasks. For fine-tuning, the BERT model is first initialized with the pre-trained parameters, and then all parameters are fine-tuned using labeled data from the downstream tasks. Each downstream task has a separate fine-tuned model, even though they are initialized with same pre-trained parameters.

**Model Architectures.** BERT's implementation is almost identical to the [[Transformer and LLM|original work]], and in this article, we denote the number of layers as $L$, the hidden size as $H$, and the number of self-attention heads as $A$.

**Input/Output Representations.** To make BERT handle a variety of downstream tasks, out input representation should be able to unambiguously represent both a single sentence and a pair of sentences (e.g. Question, Answer pairs) in one token sequence. In the context of this work, a "sentence" can be an arbitrary span of _contiguous text_. A "sequence" refers to the input token sequence to BERT, which may be a single sentence or multiple sentences.

In BERT, we use WordPiece embeddings with a 30,000 token vocabulary. The _first_ token of every sequence is always a special classification token `[CLS]`. And to differ sentences, we separate them with a special token `[SEP]`. Besides, we add a _learned embedding_ to every token indicating whether it belongs to sentence `A` or sentence `B`. For **denotation**, we use $E$ to denote input embedding, the final hidden vector of the special `[CLS]` as $C\in \mathbb{R}^{H}$, and the final hidden vector of the $i^{\text{th}}$ input token as $T_{i}\in \mathbb{R}^{H}$.

For a given token, its input representation is constructed by _summing_ the corresponding token, segment, and position embeddings.

## Pre-training BERT

We don't use traditional left-to-right or right-to-left language models to pre-train BERT. Instead, we pre-train BERT using two unsupervised tasks:

**Task #1: Masked LM.** In order to train a bidirectional representation, we simply mask some percentage of the input tokens at random, and then predict those masked tokens. In contrast to denoising auto-encoders, _we only predict the masked words_ rather than reconstructing the entire input.

But there is a problem of such training method that is we are creating a mismatch between pre-training and fine-tuning, since the `[MASK]` token does not appear during fine-tuning. To mitigate this, we do not always replace the 15% token to be masked with `[MASK]`, but `[MASK]` for 80%, random token for 10% and original token for 10%.

**Task #2: Next Sentence Prediction (NSP).** Many important downstream tasks are based on understanding the _relationship_ between two sentences, which is not directly captured by language modeling. In BERT, we pre-train for a binarized _next sentence prediction_ task that can be trivially generated from any monolingual corpus. Specifically, when choosing the sentences `A` and `B` for each pre-training example, 50% of the `B` is the actual next sentence that follows `A` (labeled as `IsNext`) and 50% of it is a random sentence from the corpus (labeled as `NotNext`).

## Fine-tuning BERT

Since BERT is trained with bidirectional information flow, so when fine-tuning for pair texts, we don't need to encode pair separately and do the cross attention, we can just concatenate them and do self-attention.

For each task, we simply plug in the task-specific inputs and outputs into BERT and fine-tune all parameters end-to-end.

# Conclusion

They trained on the bidirectional architecture, mainly just changing how to train Transformers, but it is their consideration to down-stream tasks, made the model performs better and easier to be transferred to more tasks.
