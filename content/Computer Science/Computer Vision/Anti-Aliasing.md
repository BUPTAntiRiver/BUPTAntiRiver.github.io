# Materials

- https://alvyray.com/Memos/CG/Microsoft/6_pixel.pdf — Alvy Ray Smith's memo arguing that a pixel is a point sample of a continuous image rather than a little square, which is the root reason aliasing exists.
- https://learnopengl.com/Advanced-OpenGL/Anti-Aliasing — A hands-on walkthrough of multisample anti-aliasing in OpenGL that shows how per-sample coverage is resolved into a final pixel color.
- https://developer.download.nvidia.com/assets/gamedev/files/sdk/11/FXAA_WhitePaper.pdf — NVIDIA's original FXAA white paper by Timothy Lottes, a single-pass post-processing filter that detects and blends edges for cheap anti-aliasing.
- https://www.iryoku.com/smaa/ — The SMAA project page from Jimenez et al., combining morphological edge detection with subpixel sampling and shipping a demo with full source.
- https://www.elopezr.com/temporal-aa-and-the-quest-for-the-holy-trail/ — A production-oriented deep dive into temporal anti-aliasing covering jitter, reprojection, history rejection, and sharpening.
