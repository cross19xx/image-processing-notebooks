# Reading the Histogram

Min-max stretching pulls the darkest and brightest values present out to 0 and 255. One dead pixel at 0 and one hot pixel at 255 defeat it, because the image already spans the full range and there is nothing to stretch. To do better you need to know where all the values sit, and that is what a histogram records.

## The ideas

### A histogram counts values

A grayscale histogram has 256 bins, one per possible value, each counting the pixels that hold it. It is blind to position. The counts always sum to `h * w`, which makes a free assertion. Three equivalent ways to get one:

```python
counts = np.bincount(gray.ravel(), minlength=256)              # fastest
counts, edges = np.histogram(gray, bins=256, range=(0, 256))  # 256, not 255
counts = cv2.calcHist([gray], [0], None, [256], [0, 256]).ravel()  # float32
```

`minlength=256` matters: without it an image whose brightest pixel is 200 gives only 201 bins.

The shapes read quickly. Mass on the left is a dark image, on the right a bright one. A narrow hump is low contrast, and a spike at 0 or 255 is clipping, detail lost to pure black or white. A histogram confined to 150-230 is a bright, hazy photo.

### The CDF and equalisation

The cumulative sum of the counts is the CDF: `cdf[v]` is the number of pixels at value `v` or darker. Equalisation uses it as the mapping, so a pixel brighter than 70% of the image goes 70% of the way up the range:

```
new(v) = round((cdf[v] - cdf_min) / (N - cdf_min) * 255)
```

Here N is the total pixel count and `cdf_min` is the CDF at the darkest value present. It is implemented as a lookup table applied to the whole image at once:

```python
counts = np.bincount(gray.ravel(), minlength=256)
cdf = counts.cumsum()
cdf_min = cdf[counts > 0][0]
lut = np.round((cdf - cdf_min) / (gray.size - cdf_min) * 255).clip(0, 255).astype(np.uint8)
equalised = lut[gray]
assert np.array_equal(equalised, cv2.equalizeHist(gray))
```

This matches `cv2.equalizeHist` exactly. Replace `np.round` with a truncating `astype` and the assertion fails, with many pixels one level low.

Unlike min-max, equalisation looks at the whole histogram, so outliers cannot stop it. On a faint blob at 100-130 with one pixel at 10 and one at 245, the stretch moves the blob almost nowhere while equalisation spreads it across more than 100 levels. The cost is that a large flat background takes a large share of the range.

### Colour and CLAHE

`cv2.equalizeHist` takes one channel. Equalising B, G and R separately gives each its own curve, the ratios between them change, and hues shift: a beige wall turns blue. Equalise only brightness instead, in a colour space that has a brightness channel:

```python
ycrcb = cv2.cvtColor(bgr, cv2.COLOR_BGR2YCrCb)
ycrcb[:, :, 0] = cv2.equalizeHist(ycrcb[:, :, 0])
out = cv2.cvtColor(ycrcb, cv2.COLOR_YCrCb2BGR)
```

Global equalisation uses one curve for the whole image, so a bright sky and a shadowed face compete for the same levels. CLAHE equalises small tiles separately and caps how much any bin can be boosted, which keeps noise down: `cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8)).apply(gray)`.

## Gotchas

- Use range `[0, 256]`, not `[0, 255]`; the upper end is exclusive.
- Equalisation never splits a bin, so pixels that were equal stay equal.
- Round the lookup table. Truncating is off by one against OpenCV.
- Never equalise B, G and R separately.

## Sources

- OpenCV, [Histograms 1: Find, Plot, Analyze](https://docs.opencv.org/4.x/d1/db7/tutorial_py_histogram_begins.html)
- OpenCV, [Histograms 2: Histogram Equalization](https://docs.opencv.org/4.x/d5/daf/tutorial_py_histogram_equalization.html)
- Wikipedia, [Histogram equalization](https://en.wikipedia.org/wiki/Histogram_equalization)

The brief is in BRIEF.md; the notebook here is one worked solution.
