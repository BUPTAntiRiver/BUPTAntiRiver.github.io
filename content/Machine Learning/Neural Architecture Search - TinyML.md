# What

Neural Architecture Search is a method to search for the possible structures of neural networks and reshape them into that, which can reduce latency and model size and even reach higher accuracy.

# Why

To reach to goal of tiny machine learning! This is an effective way to reduce model size and find better model structure.

# How

## Search Strategy

### Grid Search

Adjust the depth, width, channel numbers and compound scaling.

### Random Search

### Gradient Descent

### Evolutionary Search

- Mimic the evolution process in Biology.
- Fitness function: $F(\text{accuracy}, \text{efficiency})$.
- Mutation: change the depth, operators.
- Crossover: Randomly choose one operator among the two choices(from the parents) for each layer.
