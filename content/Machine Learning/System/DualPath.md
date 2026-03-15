Paper Link: http://arxiv.org/abs/2602.21548

**The Problem** they try to solve in this paper is that _the performance of multi-turn, agentic LLM is increasingly dominated by [[KV Cache|KV-cache]] storage IO_ rather than computation. Classic IO bound problem in modern development.

In prevalent [[DistServe - PD Disaggregate|dis-aggregated]] architectures, we have a prefill engine and a decode engine, therefore loading the massive KV-cache from external storage creates a **imbalance**: storage NICs (network interface card) on prefill engines become _bandwidth-saturated_, while those on decoding engines remain _idle_.

So they present DualPath, which literally adds a new path, that is storage to decode to prefill. In this path, the KV-cache is loaded into decoding engines and efficiently transferred to prefill engines via RDMA over the compute network.

# Introduction

The real world scenario is the wide spread of multi-turn agent, and inference heavy workload. There is also a huge shift in LLM inference workloads: from traditional human-LLM interaction to human-LLM-environment interaction, called the _agentic paradigm_.

Though in each individual tool call or feedback is short (often hundreds of tokens), the context accumulates across turns and can grow to extreme lengths. As a result, the workload will be highly IO bound, very high KV-cache hit rate.

Existing approaches to improve throughput under agentic workloads have converged on a common set of architectural patterns: _layer-wise prefill_, _prefill-decode (PD) disaggregation_, and _external KV-cache storage_. The problem is that _prefill-side storage network bandwidth_ becomes the bottleneck of the entire system, while decoding engines often have substantial unused storage network bandwidth.

In this paper, they propose a new method that utilizes idle decoding engines' bandwidth and use it to help prefilling via high-performance RDMA between them.

The challenges are:

1. This is a completely new path, so complex traffic managing patterns shall be introduced and may cause performance degradation if done poorly.
2. The system must decide online which loading path to use under dynamic and heterogeneous workloads, and ensure load balance across both GPUs and NICs.

## Summary

In summary, this paper makes three contributions:

- We identify the I/O-bound nature of multi-turn, agentic LLM workloads and show that **_KV-Cache loading dominates_** system performance under modern LLM inference architectures.
- We present DualPath, an inference system that introduces dual-path KV-Cache loading and **_leverages decoding-engine bandwidth_** to resolve prefill-side bottlenecks.
- We design and evaluate a workload-aware scheduling algorithm that dynamically balances computation and network resources, significantly improving balance on realistic workloads.

# Background

## LLM Inference Preliminary

The model predicts subsequent token based on previous ones, storing attention keys and values as _KV-cache_ in HBM is a good method to avoid recomputations.

**PD-disaggregated Inference.** Prefill and decode separation, the two phases exhibit distinct compute and memory patterns: _prefill_ is compute intensive and batched, while _decode_ is memory-bound and latency sensitive. Because prefill only needs to compute the attention or give out the cache hit result, and decode needs to autoregressive decoding, which needs to compute over whole sequence.

**Layerwise Prefill.** Long-context prefill is bottlenecked by HBM capacity, since if we have more KV-caches for different tokens we cannot batch them more so it lead to poor GPU utilization. Layer KV-cache performs a kind of locality, which means it will allocate KV-cache for one layer and after that free them.

## Agentic Use of LLMs

In modern agent LLMs, the prompt is usually very long, and KV-cache hit rate might be higher that 95%, and the storage requirement will be very high. In _reinforcement learning_ approaches, there is usually a _rollout_ phase, where it is prompted to generate a large number of multi-step agent trajectories. So substantial data will is offloaded to host DRAM, further constraining the available DRAM for KV-cache. This reinforces the need for external, high-capacity KV-cache storage.

## Modern AI Data Center Architecture

Their design is also compute and storage separated, in NVIDIA DGX SuperPOD their compute network and storage network are isolated from each other.

# BOTTLENECK & MOTIVATION

KV-cache loading speed is the bottleneck due to the limited bandwidth of the single storage NIC on each node. The reasons are discussed below.

First agentic workloads exhibit high KV-cache hit rates, which _require more IO and less computation_, thus creating a severe IO bottleneck. From their statistics, the mean KV-cache hit rate is 98.7%, with respect to multiple turns, long context and short appends.

Second, the _hardware evolution trend_ is not well suited for agentic inference workloads. In recent years, network bandwidth and HBM capacity have lagged behind the growth of GPU FLOPs, which drives us to run into memory and communication walls under agentic workloads.

Third, is the imbalance in storage network utilization across different engine types, which has been mentioned many times.
