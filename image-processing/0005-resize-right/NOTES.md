# Resizing images right

## Overview

This section taught me about resizing from the `cv2.resize` utility, and the three popular
interpolations used:

- Nearest (`cv2.INTER_NEAREST`)
- Bilinear interpolation (`cv2.INTER_LINEAR`)
- Cubic interpolation (`cv2.INTER_CUBIC`)
- Area Interpolation (`cv2.INTER_AREA`)

### Nearest Interpolation (`cv2.INTER_NEAREST`)

This type of interpolation calculates the target coordinates during the image manipulation and
rounds the coordinates to the nearest integer pixel. I read online that it is the fastest
interpolation method available, because it simply copies the exact pixel of the closest original
pixel without averaging any values.

```python
import cv2 as cv

test_image = cv.imread("test_image.png")  # Example size of 512x512
shrunk_image = cv.resize(
        test_image,
        dsize=None,
        fx=0.5,
        fy=0.5,
        interpolation=cv.INTER_NEAREST
)
```

### Bilinear Interpolation (`cv2.INTER_LINEAR`)

I read online that it is the default method when image manipulation in `cv2`. It calculates the
value of the new pixel by taking a weighted average of its four nearest neighbouring pixels (a 2x2
neighbourhood)

From the examples I see, It calculates the weighted average from the top two pixels, the weighted
average of the bottom two pixels, then the final weighted average of the two calculated weighted
averages. I assume this is why it's called bilinear instead of just linear.

```python
import cv2 as cv

test_image = cv.imread("test_image.png")  # Example size of 512x512
shrunk_image = cv.resize(
        test_image,
        dsize=None,
        fx=0.5,
        fy=0.5,
        interpolation=cv.INTER_LINEAR
)
```

## Numpy methods learned

### 1. `np.isin(element, test_element, assume_unique=False, invert=False, kind=None)`

It checks whether each element of a numpy array exists within the second set of comparison values.
It returns a boolean array of the same shape as your input array (first argument), where `True`
indicates the element was found and `False` indicates it was not.

#### Use Case 1: Basic filtering

```python
import numpy as np

input_array = np.array([10, 20, 30, 40, 50])

targets = np.array([20, 40, 60])

mask = np.isin(input_array, targets)
print(mask)  # Output: [False, True, False, True, False]

# Use the same mask to filter the original array
filtered = input_array[mask]
print(filtered)  # Output: [20, 40]
```

#### Use Case 2: Working with Multi-Dimensional Arrays

`np.isin` preserves the original dimensionality of the element array.

```python
import numpy as np

matrix = np.array([[1, 2], [3, 4]])

mask = np.isin(matrix, [2, 4])
print(mask)
"""
Output:
[[False, True]
 [False, True]]
"""
```

#### Use Case 3: Inverting the Output

If you want to find elements that are not present, use either the `invert=True` argument or the
bitwise NOT operator.

```python
import numpy as np

input_array = np.array([1, 2, 3, 4])

mask = np.isin(input_array, [2, 4], invert=True)
bitwise_mask = ~np.isin(input_array, [2, 4])

print(mask)
print(bitwise_mask)
"""
Both Output: [True, False, True, False]
"""
```

### 2. `np.allclose(a, b, rtol=1e-05, atol=1e-08, equal_nan=False)`

It checks whether two arrays are approximately equal element-wise within a specified tolerance. It
simply answers the question, “Are all corresponding values in these two arrays close enough to be
treated as equal?”

```python
import numpy as np

a = np.array([0.1 + 0.2, 0.5])
b = np.array([0.3, 0.5])

print(a == b)  # Output: [False  True]

print(np.allclose(a, b))  # Output: True
```

Why? Because 0.1 + 0.2 may internally be something like `0.30000000000000004`.

Conceptually, NumPy checks each pair of values using:

```text
abs(a - b) <= atol + rtol * abs(b)
```
`atol` gives you a baseline tolerance, while `rtol` lets that tolerance grow appropriately with the magnitude of the values.
