Paper link: https://arxiv.org/abs/2409.19256v2

# What?

VeRL is a large language model post training framework, which represents the whole training structure as a DAG (directed acyclic graph), the nodes in the graph are NNs (neural networks), we have many nodes in the graph is because RL in LLM has actor, critic, reward model and reference policy, they can all be distinct models. And the edges represents the data dependencies.

Such graph, or we can say dataflow model is the core system level idea of VeRL.

# Why?

# How?
