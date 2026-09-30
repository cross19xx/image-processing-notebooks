# Practice: Mix the Channels

## Goal

Handle colour images without ever getting the channel order wrong: BGR vs RGB, which weight
goes with which channel, and where alpha lives. That's the skill from
[Lesson 2](LESSON.md). Task 5 also needs Lesson 1's
coordinates.

## Dataset

Both files are in this folder. Load them with relative paths and don't re-save them.

- **`astronaut.png`**: the full-colour scikit-image `astronaut` photo (NASA, public domain),
  512 × 512.
- **`badge.png`**: a 96 × 96 **RGBA** badge. The outside is fully transparent, the edge is
  anti-aliased (partial alpha), and the centre is half transparent.

## Requirements

Your notebook must:

1. **Show the BGR mistake, then fix it.** Load `astronaut.png` with plain `cv2.imread` and
   print its shape and dtype. Show it three ways side by side: straight into `plt.imshow`,
   converted with `cv2.cvtColor`, and converted by slicing. `assert` that the two conversions
   are equal.
2. **Split the channels.** Using numpy indexing (not `cv2.split`), show the R, G and B channels
   as three grayscale images with `cmap="gray", vmin=0, vmax=255`, each titled with its
   colour. In one markdown sentence, say which channel is brightest on the orange suit and why.
3. **Grayscale by hand.** Compute a grayscale image from the **BGR** array with OpenCV's
   weights, in floats, with no loops. Compare it with `cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)`:
   print the maximum absolute difference and `assert` it is at most 1. (Careful: subtracting
   two uint8 arrays wraps too.)
4. **Weighted vs plain average.** Compute the plain channel average `(B + G + R) / 3` without
   overflow, and show it next to your weighted gray. Print the pixel location, as `(row, col)`,
   where the two differ most, plus both values there, and say in one sentence why that pixel
   differs so much.
5. **Composite the badge.** Load `badge.png` so you keep its alpha, and print its shape. Blend
   it **over** the astronaut with its top-left corner at the OpenCV point `(x, y) = (392, 24)`,
   using `out = α · fg + (1 − α) · bg` in floats with `α` scaled to 0–1. Work on a copy. The
   result must be `uint8` with the photo's original shape (assert both). Show it in RGB.
6. **Show why alpha matters.** Next to your composite, show the naive paste:
   `img[rows, cols] = badge[:, :, :3]`. In one sentence, describe what went wrong.

## Format

- **A Jupyter notebook**: `mix_the_channels.ipynb` in this folder.
- Python with `numpy`, `opencv-python` and `matplotlib`.
- numpy does the work. OpenCV is for loading, converting, and checking your answers.
- **Run every cell and save the notebook with its outputs showing.** I grade from the saved
  file.
- Add a short **`NOTES.md`** here: what you built, how to run it, and where you got stuck.

## Rubric (10 points)

| Criteria                                                                     | Points |
| ---------------------------------------------------------------------------- | -----: |
| BGR shown wrong, then fixed two ways; conversions asserted equal             |      2 |
| Channels split by indexing, displayed correctly; suit channel explained      |      1 |
| Hand grayscale uses BGR-ordered weights in floats; diff ≤ 1 asserted         |      2 |
| Plain average without overflow; biggest difference located and explained     |      2 |
| Badge loaded with alpha; composite placed at (x, y) = (392, 24), not swapped |      2 |
| Naive paste shown and its failure explained                                  |      1 |

Every check here is an `assert`, not a comparison by eye. That was the one fix from Practice 1.

**Stretch (not graded, but tell me if you tried it):** write `composite(bg, fg_bgra, x, y)` as a
function that also handles a badge hanging partly off the image's right or bottom edge. Clip
both arrays to the overlap, and prove it with `x = 460`.

When your notebook is saved with outputs, tell me it's ready and I'll score it.
