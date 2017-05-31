**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a processing pipeline capable of identifying  roadway lane lines from vehicle-mounted camera images
* Describe how the pipeline works, any identifiable difficulties, and possible improvements.


---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The pipeline was implemented as a series of steps:
* Conversion of the image to grayscale
* Application of a Gaussian Blur
* Processing with the Canny edge detection algorithm
* Identification of an appropriate focus region
* Application of the Hough line identification algorithm over the focus region
* Processing of the identified lines into those identified with left- and right-boundaries
of the lane
* Construction of a single mixture line representing the *average* of the lines for each boundary, extrapolated to the edges of the focus region
* Drawing of the mixture lines onto the image

The conversion to grayscale, Gaussion blur, Canny edge detection and Hough lines algorithms are all supplied by the OpenCV v3.2 software package.

Some modification of the provided functions was necessary to pass additional parameters through to various functions.  These additional parameters were all treated as optional named parameters to avoid alterations to the function signatures.

#### Identification of focus region
I chose a two-parameter method of selecting the focus region.  Once chosen, this region is fixed for all images/frames processed. The two parameters specified are a = fraction of width at the top of the region, and b = the fraction of the image height to consider.
This produces an isosceles trapezoid extending and narrowing upward from the bottom centre of the image.

#### Processing of Hough lines into left and right groups
This was done by first verifying that both ends of a given line were on the same side of a vertical line through the centre of the image.  The sign of the vector cross-product employed tells us whether the line is on the right or left.

This approach also permits us to ignore any lines that cross the vertical center, and permits also a simple way to remove lines which are not sufficiently parallel to vertical.
This is done by comparing the dot product of the test line with the vertical line, and discarding any for which this result is less that 30% of the two-norms of the two lines.
This results in discarding any line appearing at more than ~78 degrees to vertical.

#### Construction of the mixture line
Once we have one or more lines in a line group, then we form the *average* or *mixture* line by calculating the slope and intercept for each component line, and extrapolating the horizontal coordinates for each of the upper- and lower-boundary of the focus region.
This produces a series of upper and lower horizontal coordinates, which are then averaged to produce upper and lower endpoints for the line.

There is no averaging over vertical coordinates because these are fixed at the top and bottom of the focus region.

### 2. Potential shortcomings:
As it stands, the pipeline can produce lines with significant jitter in videos, due to noise in the focus region.  Seems to work reasonably well if the road is relatively clean, but clutter can produce spurious jumps in the image.

The strategy of rejecting lines which cross the centerline, or which are not sufficiently parallel to vertical may cause problems on slow-speed roads with tight turns.  Under these scenarios the vehicle is unlikely to be travelling at speed.

### 3. Possible improvements:
The largest improvement I can think of for videos would be to implement a momentum-based update across frames, rather than processing the sequence of frames in isolation.  Given that the vehicle is not travelling at infinite velocity, it seems reasonable to assume that the lane identifer in a particular frame is a close approximation to that for a subsequent frame.

This should result in better rejection of image clutter.
