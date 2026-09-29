# An Image Is a Grid of Numbers

To your code, a photo is a numpy array. Blurring, edge detection and the convolution layers of a neural network are all arithmetic on that grid, so the first skill is reading it: how big it is, what its numbers mean, and which number sits where. Get the coordinates wrong and every later step lands on the wrong pixel.

## The ideas

### A grayscale image is a 2-D array

Each pixel is one number, its intensity. `0` is black, `255` is white, and the values between are shades of gray. The dtype is almost always `uint8`, an unsigned 8-bit integer, which is why the range stops at 255.

```python
img = cv2.imread("photo.png", cv2.IMREAD_GRAYSCALE)
img.shape   # (384, 512): 384 rows tall, 512 columns wide
img.dtype   # uint8
```

### The origin is top-left

The first axis is rows and the second is columns, with `img[0, 0]` at the top-left corner and the vertical axis pointing down. A pixel is `img[row, col]`. Rows come first because a 2-D array is a list of rows.

OpenCV's drawing and geometry functions flip the order and take points as `(x, y)`, which is `(col, row)`. Here is every convention side by side, for a pixel at row 2, column 3 in a 7 x 10 image:

| You are writing                  | Order         | Value       |
| -------------------------------- | ------------- | ----------- |
| numpy indexing, `img[...]`       | row, col      | `img[2, 3]` |
| `img.shape`                      | height, width | `(7, 10)`   |
| OpenCV points, e.g. `cv2.circle` | x, y          | `(3, 2)`    |
| OpenCV sizes, e.g. `cv2.resize`  | width, height | `(10, 7)`   |

To circle the bright pixel at `img[120, 45]`, call `cv2.circle(img, (45, 120), 8, 255)`. There is no flip of the y axis, because the origin is at the top.

### Slicing is cropping

Slices use the same `[rows, cols]` order, so numpy does image operations for free.

```python
crop = img[1:6, 2:8]     # rows 1-5, cols 2-7, still 2-D
row3 = img[3]            # one horizontal line
col0 = img[:, 0]         # one vertical line
flipped = img[::-1]      # reverse the rows: a vertical flip
img[0:2, :] = 255        # paint the top two rows white, in place
```

A slice is a view onto the same memory, so writing into `crop` changes `img` too. Call `.copy()` when you want an independent crop.

### Displaying it

`plt.imshow(img)` on a 2-D array uses the viridis colour map and stretches the darkest value present to the bottom of the scale and the brightest to the top. A gray photo comes out green and purple, and a dim one looks full-contrast. Pin both down:

```python
plt.imshow(img, cmap="gray", vmin=0, vmax=255)
```

## Gotchas

- `img.shape` is `(height, width)` but `cv2.resize` wants `dsize=(width, height)`. Passing `img.shape` straight in turns a portrait image into a landscape one.
- A square test image hides every row/column swap. Check your code on a non-square one.
- Never flip y as `height - 1 - row`. Image rows already count down from the top.
- Editing a slice edits the original unless you copied it.

## Sources

- scikit-image, [A crash course on NumPy for images](https://scikit-image.org/docs/stable/user_guide/numpy_images.html), section 4.3 on coordinate conventions
- OpenCV, [Basic Operations on Images](https://docs.opencv.org/4.x/d3/df2/tutorial_py_basic_ops.html)
- Computerphile, [Digital Images](https://www.youtube.com/watch?v=06OHflWNCOE)

The brief is in BRIEF.md; the notebook here is one worked solution.
