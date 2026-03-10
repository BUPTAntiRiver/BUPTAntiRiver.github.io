Cross Attention is usually used in encoder and decoder architecture Transformers. When we are going to decode, we have decoder hidden states as query vector and encoder output as key and value vector, because it is the context.

So for [[Encoder and Decoder|encoder or decoder only architectures]] like BERT and GPT, we don't need the cross attention, the states of encoder or decoder act as both query and context.
