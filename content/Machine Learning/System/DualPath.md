**The Problem** they try to solve in this paper is that _the performance of multi-turn, agentic LLM is increasingly dominated by KV-cache storage IO_ rather than computation. Classic IO bound problem in modern development.

In prevalent disaggregated architectures, we have a prefill engine and a decode engine, therefore loading the massive KV-cache from external storage creates a **imbalance**: storage NICs (network interface card) on prefill engines become _bandwidth-saturated_, while those on decoding engines remain _idle_.

So they present DualPath, which literally adds a new path, that is storage to decode to prefill. In this path, the KV-cache is loaded into decoding engines and efficiently transferred to prefill engines via RDMA over the compute network.
