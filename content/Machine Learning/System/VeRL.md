Paper link: https://arxiv.org/abs/2409.19256v2

# **What is VeRL?**

VeRL (the open-source repository name for the system presented as **HybridFlow**) is a highly flexible and efficient Reinforcement Learning from Human Feedback (RLHF) framework. It is specifically designed to align Large Language Models (LLMs) with human values. The framework uniquely combines both single-controller and multi-controller paradigms into a hybrid architecture, allowing for the flexible representation and highly efficient execution of complex RLHF dataflows.

# **Why need VeRL?**

Existing RLHF and general RL frameworks struggle to efficiently and flexibly manage the massive scale of modern LLM alignment due to several core limitations:

- **Inefficiency of Single-Controller Systems:** Traditional RL systems use a centralized single controller to manage all tasks. While fine for small models, this introduces massive control dispatch overhead when applied to LLMs that require distributed computation across thousands of GPUs.
- **Inflexibility of Multi-Controller Systems:** Current state-of-the-art RLHF systems (like DeepSpeed-Chat or OpenRLHF) use a multi-controller paradigm. While fast, they deeply nest distributed computation with inter-node data communication. If a researcher wants to change the RLHF algorithm (e.g., switch from PPO to Safe-RLHF or ReMax), they must extensively rewrite the underlying synchronization code, hindering code reuse and algorithmic exploration.
- **Massive Transition Overheads:** RLHF features heterogeneous workloads (e.g., actor generation vs. actor training). Existing systems often struggle to transition the massive actor model weights between the training and generation phases, leading to significant memory redundancy and high communication delays.
- **Rigid Resource Allocation:** Existing frameworks are typically limited to a single rigid hardware placement plan (e.g., either putting all models on the exact same GPUs or isolating them completely). This causes GPU underutilization due to the unbalanced computational workloads of different models.

# **How does VeRL achieve this?**

VeRL overcomes these challenges through three primary system innovations:

**1. A Hierarchical Hybrid Programming Model** VeRL decouples the computation within a single model from the data transfer between different models.

- **Intra-node (Multi-Controller):** It encapsulates the heavy distributed computations (training, inference, generation) of individual models into primitive APIs that execute efficiently using a multi-controller setup (compatible with engines like Megatron-LM or PyTorch FSDP).
- **Inter-node (Single-Controller):** A centralized single controller coordinates the execution order and manages the complex many-to-many data resharding between the models using predefined "transfer protocols," completely abstracting this complexity away from the user.

**2. The 3D-HybridEngine** To solve the bottleneck of transitioning the actor model between its generation and training phases, VeRL introduces the 3D-HybridEngine. It allows the actor model to seamlessly use different 3D parallelism strategies (data, tensor, and pipeline parallelism) for training versus generation on the exact same set of devices. By carefully reorganizing the "micro data-parallel groups," the engine enables the generation stage to reuse training weights, achieving **zero memory redundancy** and drastically reducing the communication overhead during the transition.

**3. Auto Device Mapping** VeRL includes an automated mapping algorithm designed to find the absolute best way to deploy the RLHF dataflow across a given GPU cluster. The algorithm evaluates the workload sizes, memory capacities, and communication costs to automatically determine the optimal hardware placement (e.g., whether to colocate the actor and reference models or split them) and the best parallel strategy for each model, ensuring maximum throughput.
