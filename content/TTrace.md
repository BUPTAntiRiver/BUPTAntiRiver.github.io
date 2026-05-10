Paper link: https://arxiv.org/abs/2506.09280

Modern large language models might have a lot of silent bugs which has no explicit error signal, and the final model performance after training just is not good enough or deviates from expectation. Especially in current popular distributed training scenario, the bug has various complex forms, which is very hard to debug.

TTrace is designed for solving such kind of problems. Let's read the paper together

# Introduction

Now we are training larger models with billions of trillions of parameters, and various **distributed training** techniques have been proposed to accommodate the training workload. Supporting a customized combination of all these techniques (like data parallelism, tensor parallelism, pipeline parallelism) has led to a significant implementation complexity in distributed training systems as well as **bugs**.

Given a distributed training program, how do we tell whether it is correct? One attractive approach, which is widely adopted in the industry practice, is to do differential testing by comparing the candidate distributed implementation with a **trusted reference implementation**. Developers may rely on the alignment of loss and gradient norm curve patterns between the reference and candidate runs on a smaller scale model. However with an illustration from the author of the paper, it can take over 4,000 iterations (more than 6 hours) for two curves to show minor discrepancy. Furthermore, when the potential issue is detected this way, the developer still faces the onerous task of manually locating whereabouts of the silent bug.

So ideally, we want a systematic differential testing solution that can determine if the distributed implementation matches the reference implementation with much fewer resources like a single iteration of stochastic gradient descent. Also the testing should be more fine-grained so that one can pinpoint the whereabouts of potential silent bugs.

We have two main challenges. The first challenge is how to align the tensors between the candidate and reference implementation. In distributed training employs strategies such as tensor and pipeline parallelism to partition data, models and intermediate results across devices. These sharded tensors can be reordered and physically fragmented in multiple ways, making it **_hard to find the correspondence_** between the candidate and reference. Secondly, it is non-trivial to determine whether the candidate implementation matches the reference based on the numerical tensor values. As **_floating-point arithmetic is non-associate_** due to numerical round-off errors, mathematical equivalent operations with different computation orders in the candidate implementation can can generates different numerical results as the reference.
