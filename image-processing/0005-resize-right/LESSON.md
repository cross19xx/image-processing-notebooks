# Inventing the Pixels in Between

Resizing changes the grid itself. Enlarging a 3 x 4 image to 9 x 12 needs 108 pixels from 12, so 96 must be invented. Shrinking is the reverse: many old pixels compete for each new one. Interpolation decides how.

## The ideas

### Every output pixel looks backwards

Resizing does not push old pixels forward. For each output pixel it finds where that pixel lands in the source and reads a value there. OpenCV aligns pixel centres, clamped at the edges:

```
src = (dst + 0.5) * in_size / out_size - 0.5
```

Enlarging 4 columns to 12, output column 2 lands at 0.33, a third of the way from column 0 to column 1. There is no pixel there; interpolation supplies a value. `INTER_NEAREST` skips the centring and uses `src[floor(dst * in/out)]`.

Nearest neighbour copies one source pixel: fast, blocky, and it never produces a new value. Bilinear blends the four surrounding pixels, along x on the row above and the row below, then along y. Bicubic fits a curve through a 4 x 4 neighbourhood and is sharper, but it overshoots: a hard 0 to 255 step enlarged in float32 gives values from about -28 to 283, which uint8 then clips.

```python
def bilinear(img, out_h, out_w):
    h, w = img.shape
    ys = np.clip((np.arange(out_h) + 0.5) * h / out_h - 0.5, 0, h - 1)
    xs = np.clip((np.arange(out_w) + 0.5) * w / out_w - 0.5, 0, w - 1)
    y0, x0 = np.floor(ys).astype(int), np.floor(xs).astype(int)
    y1, x1 = np.minimum(y0 + 1, h - 1), np.minimum(x0 + 1, w - 1)
    wy, wx = (ys - y0)[:, None], (xs - x0)[None, :]
    f = img.astype(np.float32)
    top = f[y0][:, x0] * (1 - wx) + f[y0][:, x1] * wx
    bottom = f[y1][:, x0] * (1 - wx) + f[y1][:, x1] * wx
    return top * (1 - wy) + bottom * wy
```

### Shrinking and aliasing

Shrinking by 3 means each output pixel stands for a 3 x 3 block, but nearest and linear sample only a point or two in it. Detail finer than the new grid can hold then leaves a pattern that was never there: aliasing. One-pixel black and white stripes shrunk by 3 come out as full-contrast stripes three times wider, when the honest answer is mid-gray (127.5).

`INTER_AREA` averages every source pixel an output pixel covers. The OpenCV docs recommend it for shrinking, and `INTER_CUBIC` (slow) or `INTER_LINEAR` for enlarging. On those stripes it is not perfectly clean: blocks of `[0, 255, 0]` and `[255, 0, 255]` average to 85 and 170, so a faint pattern survives, though the false contrast drops from 255 levels to 85. The complete fix is to blur before shrinking.

### Masks need nearest

A segmentation mask stores class IDs, which are labels and not brightness. Resize one with `INTER_LINEAR` and a border between class 1 and class 3 grows a strip of 2s, a class that was never there. Use `INTER_NEAREST` for categories.

## Gotchas

- `dsize` is `(width, height)` but `img.shape` is `(height, width)`. Half of a `(480, 640)` image is `cv2.resize(img, (320, 240))`; or pass `dsize=None, fx=0.5, fy=0.5`.
- uint8 `INTER_LINEAR` is fixed-point, a few levels off float maths. Compare hand implementations on float32 input, where they agree to within 0.001.

## Sources

- OpenCV, [cv2.resize](https://docs.opencv.org/4.x/da/d54/group__imgproc__transform.html#ga47a974309e9102f5f08231edc7e7529d) and [imgproc.hpp](https://github.com/opencv/opencv/blob/4.x/modules/imgproc/include/opencv2/imgproc.hpp)
- Computerphile, [Resizing Images](https://www.youtube.com/watch?v=AqscP7rc8_M)
- Cornell CS5670, [image resampling lecture](https://www.cs.cornell.edu/courses/cs5670/2025sp/lectures/index.html)

The brief is in BRIEF.md; the notebook here is one worked solution.
