# Issues with recurrent models
- *Linear interaction distance*, it is hard for RNN to capture long distance information.
- *Lack of parallelization*: forward and backward passes both have **O(sequence length)** operations cannot be parallelized. Future RNN hidden states cannot be computed until all previous hidden states have been computed.