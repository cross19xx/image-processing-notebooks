# Practice: Flatten the Curve

## Goal

Read what a histogram says about an image, and fix contrast with equalisation: by hand as a
lookup table, proven identical to OpenCV, and safely on colour. That's the skill from
[Lesson 4](LESSON.md).

## Dataset

Both files are in this folder, both 300 × 450 (not square).

- **`hazy.png`**: a washed-out grayscale photo with a handful of dead (0) and hot (255)
  pixels.
- **`hazy_color.png`**: the same photo in colour.

## Requirements

Your notebook must:

1. **Three histograms, one answer.** Load `hazy.png` as grayscale. Compute its histogram with
   `np.bincount`, `np.histogram` and `cv2.calcHist`. `assert` all three are equal, and that
   the counts sum to `h * w`.
2. **Read it.** Plot the histogram with its CDF on a second y-axis (`ax.twinx()`). In two
   sentences, say what the shape tells you about the photo, and what the tiny bars at 0 and
   255 are.
3. **Watch min-max fail.** Apply `cv2.normalize(..., cv2.NORM_MINMAX)`. Print the image's min
   and max, `assert` the result is unchanged, and explain why in one sentence.
4. **Equalise by hand.** Build the 256-entry LUT from the CDF, apply it with indexing (no
   loops), and `assert np.array_equal` with `cv2.equalizeHist`. Then **prove the assert can
   fail**: build a second LUT that truncates instead of rounding, and show that the same
   comparison is `False` (print how many pixels differ). Show the original and equalised
   images, each above its own histogram and CDF.
5. **Colour without the colour shift.** On `hazy_color.png`, (a) split B, G, R, equalise
   each, and merge; (b) equalise only `Y` in YCrCb. Show the original, (a) and (b) in RGB. In
   one sentence, name a region where (a) went wrong and why.
6. **Local contrast.** Apply CLAHE (`clipLimit=2.0, tileGridSize=(8, 8)`) to the gray image.
   Show it next to your global equalisation, with their histograms. In one sentence, say what
   CLAHE did differently.

## Format

- **A Jupyter notebook**: `flatten_the_curve.ipynb` in this folder.
- Python with `numpy`, `opencv-python` and `matplotlib`.
- **Run every cell and save the notebook with its outputs showing.** I grade from the saved
  file.
- Add a short **`NOTES.md`** here: what you built, how to run it, where you got stuck, and
  any questions for me.

## Rubric (10 points)

| Criteria                                                                         | Points |
| -------------------------------------------------------------------------------- | -----: |
| Three histograms asserted equal; sum asserted                                    |      1 |
| Histogram + CDF plotted; shape and outliers read correctly                       |      1 |
| Min-max shown unchanged (asserted) and explained                                 |      1 |
| Hand LUT equalisation, no loops, `array_equal` with OpenCV                       |      2 |
| Truncating LUT shown to fail the same check                                      |      1 |
| Before/after images each with histogram and CDF                                  |      1 |
| Per-channel vs Y-only equalisation shown in RGB; the shift explained             |      2 |
| CLAHE vs global, with histograms and a correct one-line difference               |      1 |

**Stretch (not graded, but tell me if you tried it):** equalise only the pixels inside a
mask, such as the face region from Lesson 1's practice, using `calcHist`'s `mask` argument or
boolean indexing, and leave the rest of the image untouched.

When your notebook is saved with outputs, tell me it's ready and I'll score it.
