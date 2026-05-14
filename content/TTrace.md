Paper link: https://arxiv.org/abs/2506.09280

Modern large language models might have a lot of silent bugs which has no explicit error signal, and the final model performance after training just is not good enough or deviates from expectation. Especially in current popular distributed training scenario, the bug has various complex forms, which is very hard to debug.

TTrace is designed for solving such kind of problems. Let's read the paper together

# Introduction

Now we are training larger models with billions of trillions of parameters, and various **distributed training** techniques have been proposed to accommodate the training workload. Supporting a customized combination of all these techniques (like data parallelism, tensor parallelism, pipeline parallelism) has led to a significant implementation complexity in distributed training systems as well as **bugs**.

Given a distributed training program, how do we tell whether it is correct? One attractive approach, which is widely adopted in the industry practice, is to do differential testing by comparing the candidate distributed implementation with a **trusted reference implementation**. Developers may rely on the alignment of loss and gradient norm curve patterns between the reference and candidate runs on a smaller scale model. However with an illustration from the author of the paper, it can take over 4,000 iterations (more than 6 hours) for two curves to show minor discrepancy. Furthermore, when the potential issue is detected this way, the developer still faces the onerous task of manually locating whereabouts of the silent bug.

So ideally, we want a systematic differential testing solution that can determine if the distributed implementation matches the reference implementation with much fewer resources like a single iteration of stochastic gradient descent. Also the testing should be more fine-grained so that one can pinpoint the whereabouts of potential silent bugs.

We have two main challenges. The first challenge is how to align the tensors between the candidate and reference implementation. In distributed training employs strategies such as tensor and pipeline parallelism to partition data, models and intermediate results across devices. These sharded tensors can be reordered and physically fragmented in multiple ways, making it **_hard to find the correspondence_** between the candidate and reference. Secondly, it is non-trivial to determine whether the candidate implementation matches the reference based on the numerical tensor values. As **_floating-point arithmetic is non-associate_** due to numerical round-off errors, mathematical equivalent operations with different computation orders in the candidate implementation can can generates different numerical results as the reference.

# Motivations, Challenges and Their Approach

Current industry has been widely using large scale distributed training systems, so they must already has some methods to handle such silent bugs. I mean, they must have right? But their practice is actually usually ad-hoc as always.

## Industry Practice: Ad-Hoc Debugging

The predominant industry practice for training bug detection is based on differential testing that compares a distributed implementation's output to that of a simpler, trusted reference implementation. Bug what to compare? How much deviation is error is usually relying on subjective judgment. While intuitive, such ad-hoc debugging is very inefficient.

## Challenges in Verification-Based Methods

Some methods targets on proving semantic equivalence of the computation graphs generated in the distributed and single-device setting. Though it provides a proof that formally establishes the correctness of computation graph, but it is very hard to implement in production.

## How TTrace Addresses the Challenges of Ad-Hoc Debugging?

The two main challenges are:

1. How to map each intermediate tensor in the distributed system to its counterpart in the reference implementation?
2. As training involves floating point operations, how to distinguish expected numerical errors from bug-induced errors?

### Challenge #1: Mapping of Semantics

Now we have Pipeline Parallelism that partitions layers, Data Parallelism that partitions data batches, Tensor Parallelism, Sequence Parallelism and Context Parallelism that partitions tensors.

TTrace assembles tensor according to the particular parallelization strategies to compare tensors. It achieves this by building a **tensor canonical mapping** system to set up the alignment. Which is user-provided.

### Challenge #2: "Expected" Numerical Errors

How to differentiate _numerical errors from bug-induced errors_? Existing solution sidestep this by casting all tensors to higher precision like FP32 or converting all computations to compute on finite fields, thereby reducing numerical errors to make bug-induced errors more apparent. But implementing such methods for training frameworks can be difficult, since many kernel has hard-coded optimization, and if we bypass these kernels our debugging will drift from practical scenario and become meaningless.

TTrace provides a non-intrusive methods, which is an empirical numerical error tolerance estimation procedure.
