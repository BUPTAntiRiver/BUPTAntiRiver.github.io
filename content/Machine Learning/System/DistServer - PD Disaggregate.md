Paper Link: http://arxiv.org/abs/2401.09670

The first paper that introduces PD disaggregation. Which separates inference of transformer models into Prefill and Decode.

# First Pass

In LLM applications there are usually two metrics called _time to first token_ (TTFT) and _time per output token_ (TPOT), and they are corresponding to different phase of LLM inference, _prefill and decoding_. Prefill is the phase that LLM generates the first token, and decoding is the following multi-step token generation. Prefill needs to be fast, making the application more interactive. Decoding also needs to be fast, at least faster than human reading speed.

Before this paper, prefill and decode tasks are put on the same GPU machine which is hard to optimize because their resource allocation are coupled together, so the LLM providers must over provide resource, which leads to low efficiency. In their experiments, both prefill and decode latency reduced a lot when served sololy.

What DistServe do is that they disaggregated prefill and decoding phase, which enables to serve up to 7.4 times more requests or 12.6 times tighter SLO (Service Level Objectives) under various latency constraints.

# Second Pass
