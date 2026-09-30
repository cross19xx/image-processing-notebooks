# When 255 + 1 Is 0

`uint8` is a fine way to store pixels and a dangerous way to compute with them. Blurring, sharpening and edge detection all multiply and add pixel values, so you need to know what goes wrong in `uint8`, how to move into floats and back, and what people mean by "normalise".

## The ideas

### uint8 wraps; OpenCV saturates

A `uint8` holds 256 values. numpy array arithmetic stays in `uint8` by wrapping modulo 256: `250 + 10` gives `4` and `10 - 20` gives `246`, with no warning. OpenCV's `cv2.add` and `cv2.subtract` saturate instead, clamping at 255 and 0. So brightening with numpy turns the brightest pixels black.

numpy 2 adds one rule: a Python int that does not fit the dtype raises `OverflowError`. `img + 100` wraps silently, but `img + 300` and `img + (-60)` both raise. Write `img - 60` instead.

### Do the maths in floats

Leave `uint8` before computing and come back once at the end, using floats from 0 to 1:

```python
f = img.astype(np.float32) / 255
brighter = f + 0.3
out = np.clip(np.round(brighter * 255), 0, 255).astype(np.uint8)
```

Use `float32`, because plain `img / 255` gives `float64` at twice the memory. Divide by 255, because `astype` keeps the numbers: 200 becomes 200.0, not 0.78, and matplotlib clips a float RGB image above 1 to white. Clip, round, then cast, because `astype(np.uint8)` neither rescales nor clamps. With numpy 2.5, `[300.0, -5.0, 255.7]` cast directly came out as `[44, 251, 255]`: two wraps and a truncation.

16-bit files need `cv2.IMREAD_UNCHANGED` to keep their depth (plain `imread` converts them down to 8-bit), and their maximum is not 255. Read it from the dtype with `np.iinfo(img.dtype).max`, which is 65535 for `uint16`.

### Two things called "normalise"

Min-max normalisation rescales so the data's own minimum and maximum hit the ends of the range:

```
(x - x.min()) / (x.max() - x.min())
```

OpenCV's version is `cv2.normalize(img, None, 0, 255, cv2.NORM_MINMAX)`. It is what matplotlib does silently when you leave out `vmin` and `vmax`.

Standardisation subtracts a mean and divides by a standard deviation, per channel: `(x - mean) / std`. The result is centred on 0 and is no longer displayable; it is what neural networks expect. torchvision's pretrained models use the ImageNet statistics, in RGB order, after rescaling to 0-1:

```python
rgb = cv2.cvtColor(bgr, cv2.COLOR_BGR2RGB).astype(np.float32) / 255
mean = np.array([0.485, 0.456, 0.406], dtype=np.float32)
std = np.array([0.229, 0.224, 0.225], dtype=np.float32)
x = (rgb - mean) / std          # broadcasts over the channel axis
x = x.transpose(2, 0, 1)        # (C, H, W) for PyTorch
```

## Gotchas

- A BGR array from `cv2.imread` must be converted before using the RGB-ordered ImageNet statistics.
- `astype` is a cast, not a conversion. It does not scale and does not clamp.
- Truncating instead of rounding on the way back loses a level on many pixels, so a float round trip is only exact if you round.
- Hard-coding 255 as the maximum breaks on 16-bit data.
- A shape bug hides in square images: take the size with `h, w = img.shape[:2]`, never `shape[1:]`.

## Sources

- scikit-image, [Image data types and what they mean](https://scikit-image.org/docs/stable/user_guide/data_types.html)
- OpenCV, [Arithmetic Operations on Images](https://docs.opencv.org/4.x/d0/d86/tutorial_py_image_arithmetics.html)
- numpy, [Overflow errors](https://numpy.org/doc/stable/user/basics.types.html#overflow-errors)
- torchvision, [ResNet-50](https://docs.pytorch.org/vision/stable/models/generated/torchvision.models.resnet50.html)

The brief is in BRIEF.md; the notebook here is one worked solution.
