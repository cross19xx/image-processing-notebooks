# Image Processing Fundamentals

Practice from a self-built image-processing course: for each lesson, LESSON.md summarises what was taught, BRIEF.md is the assignment, and the notebook is one worked solution. The astronaut photo is from NASA (public domain), via scikit-image; the other images are generated.

| # | Lesson | Practice | What you build |
| --- | --- | --- | --- |
| 01 | [An Image Is a Grid of Numbers](0001-read-the-grid/LESSON.md) | [Read the Grid](0001-read-the-grid/BRIEF.md) | Load a real photo as a numpy array, then read, find, crop, flip and mark pixels by coordinates, with numpy first and OpenCV as the check. |
| 02 | [Three Numbers per Pixel](0002-mix-the-channels/LESSON.md) | [Mix the Channels](0002-mix-the-channels/BRIEF.md) | Fix a BGR mix-up, split and weigh colour channels by hand to match cvtColor, and alpha-composite a transparent badge onto a photo at a given (x, y). |
| 03 | [When 255 + 1 Is 0](0003-normalise-safely/LESSON.md) | [Normalise Safely](0003-normalise-safely/BRIEF.md) | Brighten a dim photo three ways, round-trip it through floats exactly, stretch and standardise it, and load a 16-bit frame without losing precision. Every claim is an assert. |
| 04 | [Reading the Histogram](0004-flatten-the-curve/LESSON.md) | [Flatten the Curve](0004-flatten-the-curve/BRIEF.md) | Read a hazy photo's histogram, watch min-max fail on two stray pixels, equalise it by hand to match OpenCV exactly, and fix the colour version without shifting its hues. |
| 05 | [Inventing the Pixels in Between](0005-resize-right/LESSON.md) | [Resize Right](0005-resize-right/BRIEF.md) | Write nearest and bilinear resizing by hand and prove them against OpenCV with tolerances in the right units, make a zone plate alias, catch bicubic overshooting, and keep a mask's classes intact. |
| 06 | [A Pixel Meets Its Neighbours](0006-slide-the-window/LESSON.md) | [Slide the Window](0006-slide-the-window/BRIEF.md) | Write cross-correlation by hand with sliding windows, prove it against filter2D with a flipped-kernel version that fails the same check, match every border mode, catch uint8 saturation, and time a separable blur. |
