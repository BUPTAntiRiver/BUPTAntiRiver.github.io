# Definition

Mathematical one: [wikipedia](https://en.wikipedia.org/wiki/Convolution)

In machine learning, convolution means we have an **input matrix**, a **convolution kernel**, a **stride**, a **pad**, and output a result. Suppose we have a $5 \times 5$ input matrix, and a $3 \times 3$ kernel with stride 1 and pad 1. Pad means we pad the input matrix to $7\times 7$ first (both top and bottom, left and right) then we traverse all $3\times 3$ part of the input matrix and do a computation defined by user with the kernel, the stride means how many elements we shift each time.
# Image-to-Column Algorithm

Besides naive element-wise multiplication and scanning through the input, is there other more efficient methods to do the convolution? Yes indeed, with `im2col` we can calculate convolution with matrix multiplication.

`Im2Col` turns the input matrix into a vector by flattening each window and stack them as columns in a matrix, then we flatten the kernel too, so the original convolution can be performed as a matrix multiplication between flat kernel and flat input.

When performing backward, we can also apply this thought, by considering input and kernel as two matrices, and so the gradients will be very easy to compute but we still need to modify the shape of matrix gradient to match it with input shape and kernel shape.
