# Practice: Resize Right

## Goal

Resize images on purpose: know where each output pixel samples the source, pick the
interpolation that fits the job, and recognise aliasing when you see it. That's the skill from
[Lesson 5](LESSON.md).

## Dataset

All three files are in this folder, and none is square.

- **`zone_plate.png`**: 400 × 600 grayscale rings that get finer towards the edges, a standard
  aliasing test.
- **`face_small.png`**: a tiny 48 × 56 colour crop of the astronaut's face.
- **`mask.png`**: a 120 × 180 segmentation mask whose pixels are class IDs `0`, `1` and `3`.
  Load it with `cv2.IMREAD_UNCHANGED`.

## Requirements

Your notebook must:

1. **Size it right.** Resize `zone_plate.png` to 150 wide and 100 tall once with `dsize`, and
   once with `dsize=None, fx=..., fy=...`. `assert` both shapes are `(100, 150)`.
2. **Nearest by hand.** Write `nearest(img, out_h, out_w)` using OpenCV's
   `floor(dst * scale)` rule, vectorised with index arrays (no loops over pixels). On the gray
   face, `assert np.array_equal` with `cv2.INTER_NEAREST` for one enlargement by a non-integer
   factor (e.g. 107 × 131) and one shrink.
3. **Bilinear by hand.** Write `bilinear(img, out_h, out_w)` with half-pixel centres and edge
   clamping, in float32. Compare it with `cv2.resize` on the **float32** image using
   `np.allclose`, with a tolerance you choose and justify in one sentence *in gray levels*.
   Then show the assert can fail: a version without the `+ 0.5 ... - 0.5` centring must break
   it (print its max difference). Finally print the max difference against OpenCV's **uint8**
   path and say in one sentence why it isn't zero.
4. **Make it alias.** Shrink the zone plate to 100 × 150 with `INTER_NEAREST`, `INTER_LINEAR`
   and `INTER_AREA`, and show all three. Print each result's mean and standard deviation, and
   `assert` that `INTER_AREA` has the lowest standard deviation. In two sentences, say where
   the false rings appear and why there, and what the lower spread is telling you.
5. **Catch the overshoot.** Enlarge the face ×6 with nearest, linear and cubic, and show them
   in RGB. Run cubic on the float32 image as well. Print how many values land below 0 or above
   255 and `assert` there are some. In one sentence, say what that does to a uint8 result.
6. **Keep the classes.** Resize the mask to 84 × 126 with `INTER_LINEAR` and `INTER_NEAREST`.
   `assert` the nearest result's classes are exactly `{0, 1, 3}`, and that the linear result
   contains a value outside that set. Print the invented values and show *where* they are (for
   example, as a highlighted mask).

## Format

- **A Jupyter notebook**: `resize_right.ipynb` in this folder.
- Python with `numpy`, `opencv-python` and `matplotlib`.
- **Run every cell, in order, and save the notebook with its outputs showing.** I grade from
  the saved file, and I re-run it from the top.
- **`NOTES.md`** in this folder: what you built, how to run it, where you got stuck, and any
  questions for me. This is now graded.

## Rubric (10 points)

| Criteria                                                                            | Points |
| ----------------------------------------------------------------------------------- | -----: |
| Both resize routes asserted to `(100, 150)`                                         |      1 |
| Hand nearest, vectorised, `array_equal` with OpenCV for enlarge and shrink          |      1 |
| Hand bilinear matches OpenCV float32 with a justified tolerance in gray levels      |      2 |
| The un-centred version shown to fail; uint8 difference explained                    |      1 |
| Zone plate aliasing shown, std asserted, rings located and explained                |      2 |
| Cubic overshoot counted, asserted, and its uint8 effect explained                   |      1 |
| Mask classes preserved by nearest and broken by linear, asserted and located        |      1 |
| `NOTES.md` present                                                                  |      1 |

**Stretch (not graded, but tell me if you tried it):** blur the zone plate with
`cv2.GaussianBlur(img, (0, 0), sigmaX=1.5)` *before* shrinking it with `INTER_NEAREST`, and
compare it with your unblurred nearest result. That's a preview of why Lesson 7 matters.

When your notebook is saved with outputs, tell me it's ready and I'll score it.
