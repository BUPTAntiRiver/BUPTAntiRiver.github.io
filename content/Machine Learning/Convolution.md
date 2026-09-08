Besides naive element-wise multiplication and scanning through the input, is there other more efficient methods to do the convolution? Yes indeed, with `im2col` we can calculate convolution with matrix multiplication.

# Image-to-Column Algorithm

`Im2Col` vectorizes the input matrix by flattening each window and stack them as columns in a matrix, then we flatten the kernel too, so the original convolution can be performed as a matrix multiplication between flat kernel and flat input.
When performing backward, we can also apply this thought, by considering input and kernel as two matrices, and so the gradients will be very easy to compute but we still need to modify the shape of matrix gradient to match it with input shape and kernel shape.
