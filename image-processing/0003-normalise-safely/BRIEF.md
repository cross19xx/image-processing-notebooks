# Practice: Normalise Safely

## Goal

Move pixel values between `uint8`, floats and back without a single silent wrap, clip or
truncation, and tell min-max normalisation from standardisation. That's the skill from
[Lesson 3](LESSON.md).

## Dataset

Both files are in this folder. Load them with relative paths.

- **`dim.png`**: an underexposed, low-contrast colour photo. It is **not square**.
- **`sensor16.png`**: a single-channel **16-bit** frame, also not square.

## Requirements

Your notebook must:

1. **Load and size.** Load `dim.png` with plain `cv2.imread`. Unpack `h, w = img.shape[:2]`,
   print both with labels, and `assert` which is larger, so the assert would fail if you had
   swapped them. Print the dtype and each channel's min and max.
2. **Brighten three ways**, all by `k = 160`: numpy `img + 160`, `cv2.add`, and the float path
   (to 0–1 `float32`, add, then clip, round, cast). `assert` the OpenCV and float results are
   identical. Print how many values the numpy version got wrong, and show all three **in RGB**
   side by side.
3. **Round trip.** Convert `img` to `float32` in 0–1. `assert` the dtype, that the min is ≥ 0
   and the max is ≤ 1. Convert back with clip, round, cast, and `assert np.array_equal` with
   the original.
4. **Min-max stretch.** Stretch the image to the full 0–255 range by hand, in floats, using
   the whole image's min and max. Compare it with
   `cv2.normalize(img, None, 0, 255, cv2.NORM_MINMAX)`, and `assert` the maximum absolute
   difference is ≤ 1 (compute that difference in a dtype that can't wrap). Show the before and
   after in RGB.
5. **Standardise twice.** (a) With ImageNet's statistics, `mean=[0.485, 0.456, 0.406]` and
   `std=[0.229, 0.224, 0.225]`, applied the way the lesson describes (mind the channel order).
   Print the result's per-channel mean and std, and explain in one sentence why they aren't 0
   and 1. (b) With this image's own per-channel mean and std. `assert` with `np.allclose`
   that every channel now has mean ≈ 0 and std ≈ 1.
6. **16-bit.** Load `sensor16.png` twice: plain `cv2.imread` and `cv2.IMREAD_UNCHANGED`.
   Print each one's shape, dtype and max. Normalise the 16-bit version to 0–1 using
   `np.iinfo`, not a hard-coded number, and `assert` its max is ≤ 1. In one sentence, say what
   plain `imread` threw away.

## Format

- **A Jupyter notebook**: `normalise_safely.ipynb` in this folder.
- Python with `numpy`, `opencv-python` and `matplotlib`.
- **Run every cell and save the notebook with its outputs showing.** I grade from the saved
  file.
- Add a short **`NOTES.md`** here: what you built, how to run it, and where you got stuck.

## Rubric (10 points)

| Criteria                                                                          | Points |
| --------------------------------------------------------------------------------- | -----: |
| Size unpacked height-first, with an assert that catches a swap                    |      1 |
| Three brightens; OpenCV == float path asserted; wrapped count printed; RGB display |      2 |
| Float round trip with dtype/range asserts and exact `array_equal`                 |      2 |
| Min-max by hand matches `cv2.normalize` (≤ 1, no wrap in the diff)                |      1 |
| ImageNet standardisation in RGB order, with the "why not 0/1" explained           |      1 |
| Own-stats standardisation with `np.allclose` asserts                              |      1 |
| 16-bit loaded both ways, normalised via `np.iinfo`, the loss explained            |      1 |
| **Every check is an `assert`**, and images are shown in RGB                       |      1 |

**Stretch (not graded, but tell me if you tried it):** turn your standardised image from 5(b)
back into a displayable `uint8` image, and explain why min-max is the right tool for that step.

When your notebook is saved with outputs, tell me it's ready and I'll score it.
