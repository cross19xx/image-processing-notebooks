# Image processing notebooks

Practice notebooks from a course on classical image processing I'm working through: pixels as
arrays, colour channels, dtypes and normalisation, histograms, resizing, and (as the course
continues) kernels, convolution, blur, sharpening, Sobel and Canny. Every operation is built by
hand in numpy first, then checked against OpenCV.

Each lesson has its own folder under `image-processing/`. `LESSON.md` summarises what the lesson
taught, `BRIEF.md` is the assignment, the notebook is my worked solution with its outputs saved,
and `NOTES.md`, where present, holds what I learned and where I got stuck. The data files each
notebook needs sit next to it. The index of lessons is in `image-processing/README.md`.

To run a notebook:

    python3 -m venv .venv && source .venv/bin/activate
    pip install numpy opencv-python matplotlib jupyter
    jupyter lab image-processing

Notebooks load their data with relative paths, so start Jupyter from the folder above or open
the notebook from its own folder.
