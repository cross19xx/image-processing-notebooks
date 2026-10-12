# A Pixel Meets Its Neighbours

Every operation up to this point treated pixels one at a time. Normalisation rescaled each
value, equalisation remapped each value through a lookup table, and resizing moved pixels
without comparing them. Filtering is different: the output at a pixel depends on the pixels
around it. The mechanism is a kernel, a small grid of weights slid across the image, and it is
the same operation whether the goal is a blur, a sharpening, an edge map, or a layer inside a
convolutional network.

## The ideas

### The slide-and-sum

A kernel is a small matrix, usually 3 x 3 or 5 x 5. For each output pixel, centre the kernel
on that pixel, multiply each weight by the pixel under it, and add the products:

    out[r, c] = sum over i, j of k[i, j] * img[r + i - 1, c + j - 1]

Four kernels show the range of behaviour. The identity (a single 1 at the centre) returns the
image unchanged. A single 1 at the middle-right position reads each pixel's right neighbour, so
the whole image shifts left by one pixel. The box kernel, nine copies of 1/9, replaces each
pixel with the mean of its neighbourhood and turns a hard edge into a ramp. The horizontal
difference kernel, `[-1, 0, 1]` across the middle row, returns 0 on flat regions and a signed
response at edges: positive where the image gets brighter to the right, negative where it
gets darker.

### Correlation and convolution

The operation above is cross-correlation: the kernel is applied as drawn. Convolution flips
the kernel in both axes first. For symmetric kernels the two are identical, which is why the
words get used interchangeably. For an asymmetric kernel they differ: as correlation the
"pick right neighbour" kernel shifts the image left; as convolution it shifts it right.

`cv2.filter2D` computes correlation. Its documentation says so directly: "The function does
actually compute correlation, not the convolution." `scipy.ndimage.convolve` does flip. The
"convolution" layers in deep-learning frameworks are cross-correlation too; since a network
learns its kernels, the distinction does not matter there.

### Borders

At the edge the window hangs off the image, so the image is padded first. The mode matters:

| OpenCV flag          | np.pad mode   | Border filled with                        |
| -------------------- | ------------- | ----------------------------------------- |
| `BORDER_CONSTANT`    | `'constant'`  | zeros, which darkens a blurred edge       |
| `BORDER_REPLICATE`   | `'edge'`      | the nearest edge pixel                    |
| `BORDER_REFLECT_101` | `'reflect'`   | a mirror that does not repeat the edge pixel (OpenCV's default) |
| `BORDER_REFLECT`     | `'symmetric'` | a mirror that repeats the edge pixel      |

Note the naming mismatch: numpy's `'reflect'` is OpenCV's reflect-101, and OpenCV's plain
`BORDER_REFLECT` is numpy's `'symmetric'`.

### By hand in numpy

```python
import numpy as np, cv2
from numpy.lib.stride_tricks import sliding_window_view

def correlate(img, kernel, mode="reflect"):
    pad = kernel.shape[0] // 2
    padded = np.pad(img.astype(np.float32), pad, mode=mode)
    windows = sliding_window_view(padded, kernel.shape)   # (h, w, kh, kw)
    return np.einsum("ijkl,kl->ij", windows, kernel.astype(np.float32))

box = np.ones((3, 3), np.float32) / 9
assert np.allclose(correlate(gray, box), cv2.filter2D(gray, cv2.CV_32F, box), atol=1e-3)
```

### Separable kernels

A k x k kernel costs k squared multiply-adds per pixel. When a kernel equals a column vector
times a row vector, two 1-D passes give the same result for 2k operations. The box kernel is
separable (`ones(3, 3) / 9` is the outer product of `[1, 1, 1] / 3` with itself), and so is
the Gaussian. On a 320 x 480 image a 5 x 5 box took 3.3 ms as one 2-D pass and 0.7 ms as two
1-D passes; at 15 x 15 the gap was 12.3 ms against 1.0 ms. OpenCV exposes the two-pass form as
`cv2.sepFilter2D`.

## Gotchas

- `ddepth=-1` keeps the input's uint8, so the output is rounded and saturated. A 3 x 3
  all-ones kernel pushed 131,043 of 153,600 pixels past 255, and all came back as 255. Worse,
  the difference kernel's negative responses are clipped to 0 and vanish. Use `cv2.CV_32F`
  for any kernel with negative weights or a sum above 1.
- Zero padding darkens the outer ring of a blur. `filter2D`'s default, reflect-101, does not.
- An asymmetric kernel is applied as drawn by `filter2D` and flipped by `scipy.ndimage.convolve`;
  the two outputs differ by a shift.

## Sources

- Shree Nayar, Image Processing I (First Principles of Computer Vision): https://cave.cs.columbia.edu/Statics/monographs/Image%20Processing%201%20FPCV-1-4.pdf
- OpenCV `filter2D` reference: https://docs.opencv.org/4.x/d4/d86/group__imgproc__filter.html#ga27c049795ce870216ddfb366086b5a04
- OpenCV tutorial, Smoothing Images: https://docs.opencv.org/4.x/d4/d13/tutorial_py_filtering.html
- 3Blue1Brown, But what is a convolution?: https://www.youtube.com/watch?v=KuXjwB4LzSA
- Victor Powell, Image Kernels explained visually: https://setosa.io/ev/image-kernels/

The brief is in BRIEF.md; the notebook here is one worked solution.
