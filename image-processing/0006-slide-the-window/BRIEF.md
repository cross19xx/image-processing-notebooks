# Practice: Slide the Window

## Goal

Own the slide-and-sum: implement it without loops, know which way an asymmetric kernel
moves things, pad the border on purpose, keep the output dtype honest, and split a kernel
into two 1-D passes. That's the skill from
[Lesson 6](LESSON.md).

## Dataset

**`portrait.png`** is in this folder: a 320 × 480 grayscale crop (not square). Load it with
`cv2.IMREAD_GRAYSCALE`.

## Requirements

Each assert below must name, in a comment, the wrong answer it would catch. An assert that
can't fail scores nothing.

1. **Size.** `h, w = gray.shape` and `assert (h, w) == (320, 480)`.
2. **Correlate by hand.** Write `correlate(img, kernel, mode="reflect")` using `np.pad` and
   `sliding_window_view` + `einsum` (no loops over pixels), returning `float32`. Compare it
   with `cv2.filter2D(gray, cv2.CV_32F, k)` for a 5 × 5 box kernel **and** for the asymmetric
   kernel `[[0,0,0],[0,0,1],[0,0,0]]`, each with `np.allclose` and an `atol` you state in gray
   levels. Then **prove the assert can fail**: run the same comparison with the kernel flipped
   (`k[::-1, ::-1]`, i.e. true convolution). Print its max difference and `assert` the
   comparison is `False`. In one sentence, say why the flipped box kernel would *not* have
   failed.
3. **Borders.** Extend `correlate` so `mode` maps to each of `cv2.BORDER_CONSTANT`,
   `BORDER_REPLICATE`, `BORDER_REFLECT_101` and `BORDER_REFLECT`, and `assert` each matches
   `filter2D` with that flag on the box kernel. Show the top-left 8 × 8 of the zero-border and
   reflect-101 results side by side and name the artefact.
4. **The dtype trap.** Run a 3 × 3 all-ones kernel with `ddepth=-1` and with `cv2.CV_32F`.
   Count the float pixels above 255 and `assert` the uint8 result equals
   `np.clip(np.round(float_result), 0, 255)` exactly. Then run the difference kernel
   `[[0,0,0],[-1,0,1],[0,0,0]]` both ways, show both, and explain in one sentence what the
   uint8 version lost.
5. **Separable.** Write `correlate_separable(img, v)` as two 1-D passes (columns, then rows)
   and `assert np.allclose` with the 2-D box of the same size. Time both with `timeit` for
   5 × 5 and 15 × 15 and print the speed-ups. `assert` the 15 × 15 separable version is faster.
   Check `cv2.sepFilter2D` agrees too.
6. **Kernel zoo.** Apply identity, shift-left, box, and the difference kernel to the portrait
   (float path), show all four, and `assert` the identity output equals the input exactly.

## Format

- **A Jupyter notebook**: `slide_the_window.ipynb` in this folder.
- Python with `numpy`, `opencv-python` and `matplotlib`.
- **Run every cell, in order, and save the notebook with its outputs showing.** I re-run it
  from the top.
- **`NOTES.md`** in this folder: what you built, how to run it, where you got stuck, and any
  questions for me. Graded.

## Rubric (10 points)

| Criteria                                                                            | Points |
| ----------------------------------------------------------------------------------- | -----: |
| Shape asserted as `(320, 480)`                                                      |    0.5 |
| Hand `correlate` matches `filter2D` on box and asymmetric kernels, tolerance stated |      2 |
| Flipped kernel shown to fail; the box exception explained                           |      1 |
| All four border modes matched; artefact named                                       |    1.5 |
| uint8 saturation counted and asserted; difference-kernel loss explained             |    1.5 |
| Separable version matches, timed, asserted faster; sepFilter2D agrees               |      2 |
| Kernel zoo shown; identity asserted exact                                           |    0.5 |
| Every assert's comment names the wrong answer it catches; `NOTES.md` present        |      1 |

**Stretch (not graded, but tell me if you tried it):** write `convolve` as `correlate` with a
flipped kernel and `assert` it matches `scipy.ndimage.convolve(img, k, mode="reflect")` on
the asymmetric kernel. Note which `scipy` mode corresponds to OpenCV's default.

When your notebook is saved with outputs, tell me it's ready and I'll score it.
