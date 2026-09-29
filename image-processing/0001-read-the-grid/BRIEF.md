# Practice: Read the Grid

## Goal

Move around a real image by coordinates without ever mixing up `(row, col)` and `(x, y)`.
That's the skill from [Lesson 1](LESSON.md), and every later
lesson in this course depends on it.

## Dataset

**`astronaut_gray.png`** is in this folder. It's the scikit-image `astronaut` photo (Eileen
Collins, NASA, public domain), converted to grayscale and cropped so it isn't square. Load
it with a relative path. Don't resize or re-save it.

## Requirements

Your notebook must:

1. **Load** the image with `cv2.imread(..., cv2.IMREAD_GRAYSCALE)`. Print its `shape`,
   `dtype`, `min()` and `max()`, then write one markdown sentence giving its height and width
   in pixels.
2. **Display** it twice, side by side: once with a bare `plt.imshow(img)` and once with
   `cmap="gray", vmin=0, vmax=255`. Say in one sentence what the bare version got wrong.
3. **Read two pixels** and print their values: the numpy index `img[100, 200]`, and the pixel
   OpenCV would call the point `(50, 300)`.
4. **Find the brightest pixel** with `np.argmax` and `np.unravel_index`. Print its location
   both as `(row, col)` and as `(x, y)`. Many pixels share the maximum value; explain in one
   sentence why `argmax` returns this particular one.
5. **Mark it.** On a *copy* of the image, draw a circle of radius 12 around that pixel with
   `cv2.circle`, and show the result. The circle must actually surround the pixel.
6. **Crop the face.** The face sits inside the box from x = 150 to x = 300 and y = 40 to
   y = 200, **both ends included**. Crop it with numpy slicing, print the crop's shape, and
   show it.
7. **Flip** the image upside down and left-to-right using slicing only. Check each against
   `cv2.flip` with `assert np.array_equal(...)`.
8. **Build an image from scratch:** a `uint8` image 100 rows tall and 256 columns wide where
   every pixel's value equals its column index (a left-to-right gradient). No loops. Assert
   `grad[42, 200] == 200` and show it. Then halve it with `cv2.resize` and assert the result's
   shape is `(50, 128)`.

## Format

- **A Jupyter notebook**: `read_the_grid.ipynb` in this folder.
- Python with `numpy`, `opencv-python` (`pip install opencv-python`) and `matplotlib`.
- Do each task in numpy first. OpenCV is for loading, drawing, and checking your answers.
- **Run every cell and save the notebook with its outputs showing.** I grade from the saved
  file.
- Add a short **`NOTES.md`** here: what you built, how to run it, and where you got stuck.

## Rubric (10 points)

| Criteria                                                                 | Points |
| ------------------------------------------------------------------------ | -----: |
| Loaded correctly; shape read out as height and width in words            |      1 |
| Display comparison done, with the difference explained                   |      1 |
| Both pixel reads correct (the `(x, y)` one is the test)                  |      2 |
| Brightest pixel located and reported in both orders; tie-break explained |      2 |
| Circle drawn around the right pixel                                      |      1 |
| Crop has the right shape, so the inclusive end was handled               |      1 |
| Flips by slicing, verified with `cv2.flip`                               |      1 |
| Gradient built without loops; resize given the right `dsize` order       |      1 |

**Stretch (not graded, but tell me if you tried it):** paint a filled white rectangle from
`(x, y) = (20, 30)` to `(60, 90)` using slicing alone, and prove it matches
`cv2.rectangle(copy, (20, 30), (60, 90), 255, -1)` pixel for pixel with `np.array_equal`.

When your notebook is saved with outputs, tell me it's ready and I'll score it.
