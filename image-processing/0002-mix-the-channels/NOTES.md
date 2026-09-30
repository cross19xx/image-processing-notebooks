# Mixing the channels

- This lesson introduces the third dimension, colour in a `cv2.imread` call
- This changes the shape into a `(w, h, c)`, where 3 is either 3 channels or 4 (if we explicitly call `IMREAD_UNCHANGED`)

## A few notes
- `cv2.imread` returns an image as BGR
- The datatype for the colour channels is `uint8` since it represent its value as 0-255

## Questions for the agent
- What is the purpose of the Rec.601? Is it to convert an image to colour or (in my example), convert to grayscale
- Do I need to continuously use `image.copy` to create copies or can I just work with the initial object read from `cv2.imread`?

## Personal research topic
- `numpy` slicing