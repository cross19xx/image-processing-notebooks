# Reading the Grid (notes)

- This lesson teaches about how to load an image into a `numpy` array using the `cv2.imread`
  function.
- It also teaches about manipulating the `ndarray` to crop.

### `numpy` methods learned

- `np.argmax`: returns the indices of the maximum values in an array, based on an axis
- `np.unravel_index`: in the application of our system, it returns the row and column based on an
  index (which we got from `argmax`) based on an n-array shape
- `ndarray.copy`: clone an ndarray
- Numpy array slicing
- `np.fliplr`: Simple horizontal flipping. An alternative may have been `np.flip(arr, 1)`

### Notes

- With the stretch goal, I need insight on whether I did it properly or not
