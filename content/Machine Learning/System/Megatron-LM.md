# Megatron-LM

Megatron-LM is a large-scale training framework developed by NVIDIA for training transformer-based language models. It is designed to efficiently train models with billions or even trillions of parameters across thousands of GPUs.

The framework is built around **Model Parallelism**, specifically combining **Tensor Parallelism** and **Pipeline Parallelism** to distribute both the compute and memory requirements of large models.

# Core Parallelism Strategies

## Tensor Parallelism (TP)

In Megatron-LM, tensor parallelism splits the weights of each transformer layer across multiple GPUs. For a linear layer $Y = XW$, the weight matrix $W$ is split along the column or row dimension:

- **Column Parallel**: $W = [W_1, W_2, \dots, W_n]$, each GPU computes $Y_i = XW_i$
- **Row Parallel**: Input $X$ is split, each GPU computes partial sums, then reduced

For attention layers, the query/key/value heads are distributed across GPUs, allowing each GPU to handle only a subset of attention heads.

## Pipeline Parallelism (PP)

The model layers are divided into stages, with each stage assigned to different GPUs. To improve GPU utilization, the input batch is split into **micro-batches** that flow through the pipeline with **1F1B** (one forward, one backward) scheduling to reduce bubble time.

## Sequence Parallelism (SP)

Megatron-LM also introduces sequence parallelism to handle long sequences. For operations like LayerNorm and Dropout that are typically computed independently per token, the sequence dimension is partitioned across GPUs, reducing activation memory.

# FSDP: Fully Sharded Data Parallel

FSDP (implemented in PyTorch as `torch.distributed.fsdp`) is based on the **ZeRO** (Zero Redundancy Optimizer) family, specifically ZeRO-3. Unlike Megatron-LM's model parallelism, FSDP is a form of **Data Parallelism**:

- Model parameters, gradients, and optimizer states are **sharded** across all GPUs
- Each GPU only stores a portion of the model parameters
- During forward/backward, parameters are **All-Gathered** just-in-time, then discarded

# Key Differences

| Aspect                      | Megatron-LM                                         | FSDP                                                   |
| --------------------------- | --------------------------------------------------- | ------------------------------------------------------ |
| **Parallelism Type**        | Model Parallelism (Tensor + Pipeline)               | Data Parallelism (Sharded)                             |
| **Communication Pattern**   | All-Reduce within layers (TP), P2P (PP)             | All-Gather, Reduce-Scatter                             |
| **Model Awareness**         | Requires model architecture integration             | Plug-and-play, wrapper-based                           |
| **Communication Frequency** | Very high (every layer for TP)                      | Lower (per micro-batch)                                |
| **Best For**                | Models that don't fit single GPU even with sharding | Scaling across many GPUs when model fits with sharding |
| **Memory Efficiency**       | Splits both activations and parameters              | Shards parameters, gradients, optimizer states         |

## When to Use Which?

**Use Megatron-LM when**:

- Training very large models (100B+ parameters) that require model parallelism
- You need maximum compute efficiency on NVIDIA hardware
- You can modify your model to support tensor parallelism

**Use FSDP when**:

- Your model can fit on a single GPU with parameter sharding
- You want a simpler setup without model architecture changes
- You need to scale across many GPUs with good memory efficiency

In practice, these approaches can be **combined**: use FSDP for data parallelism across nodes, and Megatron-LM's tensor/pipeline parallelism within each node for extremely large models.
