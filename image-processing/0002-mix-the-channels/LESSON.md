# Three Numbers per Pixel

A colour image holds three numbers per pixel: how much red, green and blue light to mix. numpy stores them on a third axis, so the shape becomes `(height, width, 3)` and indexing gains one slot, `img[row, col, channel]`. Most colour bugs come from one fact: the library that loaded the image and the library that displays it disagree about the channel order.

## The ideas

### Each channel is a grayscale image

Slicing one channel out gives a plain 2-D array, and everything true of grayscale images still holds.

```python
red = rgb[:, :, 0]     # shape (h, w)
rgb[3, 5]              # array([R, G, B]) at row 3, col 5
rgb[3, 5, 1]           # just the green value
```

Pure green (0, 255, 0) is bright in the green channel and black in the other two.

### OpenCV returns BGR

`cv2.imread` returns blue, green, red. The default flag is literally named `IMREAD_COLOR_BGR`. matplotlib, scikit-image and PIL expect RGB, so a photo passed straight from OpenCV to `plt.imshow` shows a red car as blue. Convert once, right after loading:

```python
bgr = cv2.imread("photo.png")
rgb = cv2.cvtColor(bgr, cv2.COLOR_BGR2RGB)   # a new array
rgb = bgr[:, :, ::-1]                        # same pixels, reversed view
```

The slicing version is free, but it is a view with a negative stride, and OpenCV's drawing functions refuse it with "Layout of the output array img is incompatible with cv::Mat". Wrap it in `np.ascontiguousarray(...)` before drawing.

### Alpha is a fourth channel

Alpha is opacity, not colour: `0` is transparent and `255` opaque. `cv2.imread(path)` drops it and returns `(h, w, 3)` unless you pass `cv2.IMREAD_UNCHANGED`, which gives `(h, w, 4)`. The standard "over" blend, with alpha scaled to 0-1, is

```
out = a * foreground + (1 - a) * background
```

The blend needs fractions, so do it in floats and convert back at the end. Red (220, 40, 40) over white at a = 0.5 gives (238, 148, 148).

### Three channels to one

Averaging the channels ignores that the eye finds green far brighter than blue. OpenCV uses the Rec. 601 luma weights:

```
Y = 0.299 R + 0.587 G + 0.114 B
```

They sum to 1, so white stays 255 and black stays 0. Pure green becomes 150, pure red 76 and pure blue 29. scikit-image's `rgb2gray` uses the Rec. 709 weights (0.2125, 0.7154, 0.0721) and returns floats from 0 to 1, so the two libraries give different gray images. Pick one and stay with it.

By hand, in numpy:

```python
weights = np.array([0.114, 0.587, 0.299])   # B, G, R order
gray = np.round(bgr.astype(np.float32) @ weights).astype(np.uint8)
```

This matches `cv2.cvtColor(bgr, cv2.COLOR_BGR2GRAY)` to within 1 from rounding. `cv2.COLOR_GRAY2BGR` goes back to shape `(h, w, 3)` with three identical channels, but the colour is gone for good.

## Gotchas

- Weights must follow the array's channel order. RGB-ordered weights on BGR data put some pixels 35 levels off on a test photo.
- uint8 channel sums wrap. `bgr[..., 0] + bgr[..., 1] + bgr[..., 2]` reports a maximum of 255 when the true maximum is 765. Cast to float first.
- Drawing on a reversed view fails; copy it contiguous.
- `imread` silently discards transparency.

## Sources

- OpenCV, [Color conversions](https://docs.opencv.org/4.x/de/d25/imgproc_color_conversions.html)
- OpenCV, [cvtColor reference](https://docs.opencv.org/4.x/d8/d01/group__imgproc__color__conversions.html#gaf86c09fe702ed037c03c2bc603ceab14)
- scikit-image, [rgb2gray](https://scikit-image.org/docs/stable/api/skimage.color.html#skimage.color.rgb2gray)
- Wikipedia, [Luma (video)](https://en.wikipedia.org/wiki/Luma_(video))

The brief is in BRIEF.md; the notebook here is one worked solution.
