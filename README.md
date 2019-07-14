**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[image7]: ./examples/example_output.jpg "Output"
[video1]: ./output_videos/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

### Camera Calibration

The code for this step is contained in the first three code cells of the IPython notebook located in "./examples/example.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

In second cell, I've created a function named `undistort_image` because I need to undistort every image in pipeline. This function have three inputs:

**img:** This is the input image to be distorted.

**obj_p:** This is one of the outputs of first cell: objpoints

**img_p:** This is one of the outputs of first cell: imgpoints

In this function, I used `obj_p` and `img_p` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. Then, it applies distortion correction to the input image `img` using the `cv2.undistort()` function and returns undistorted image `undistorted_img`.

In third cell, I applied this function in one of the chessboard images.

![alt text][image1]

The camera calibration seems OK, as all edges of the squares in chessboard seem to be straight and parallel. The area of squares seems to be same for all squares.

### Distortion correction of a test image

I have applied distortion correction in fourth cell on one of the test images using function `undistort_image` I have created in in second cell.

![alt text][image2]

### Creating thresholded binary image

I used a combination of color and gradient thresholds to generate a binary image.

There are 3 type of gradient thresholds that are used in this project. I created three functions for these thresholds found in cell number 5.

* Using **abs_sobel_thresh** for taking gradient at x direction and y direction
* Using **mag_thresh** for mangitude of gradient
* Using **dir_threshold** for direction of gradients

In these functions I take an input image, kernel size and minimum and maximum threshold values.(0 to 255) Then I return a binary image.
My aim is to find the gradient at line lines and eliminate unnecessary detections.

I also use color thresholds. As RGB color space is not enough to detect yellow lines, white line, and lines at shadowed images at the same time, I switched to HLS color space.

HLS is is short for Hue, Luminance and Saturation. I've created a function named **hls_select** in cell number 7 which takes inputs as image and thresholds for Hue, Luminance and Saturation.

Then I combined these thresholds to detect lane lines at different road conditions (asphalt colors, shadows etc.)

Here's an example of my output for this step.

![alt text][image3]

### Perspective Transformation

The code for my perspective transform includes a function called `perspective_transformation()`, which appears in the 12th code cell of the IPython notebook.

In this function, my goal was to get an "bird's eye view" which is essential to get the curvature of the road.

The function takes as inputs an image (`img`), and return transformed image (`warped`). I chose the hardcode the source (`src`) and destination (`dst`) points in the following manner:

```python
img_size = (img.shape[1],img.shape[0])

src = np.float32(
[[(img_size[0] / 2) - 63, img_size[1] / 2 + 100],
[((img_size[0] / 6) - 10), img_size[1]],
[(img_size[0] * 5 / 6) + 60, img_size[1]],
[(img_size[0] / 2 + 63), img_size[1] / 2 + 100]])
dst = np.float32(
[[(img_size[0] / 4), 0],
[(img_size[0] / 4), img_size[1]],
[(img_size[0] * 3 / 4), img_size[1]],
[(img_size[0] * 3 / 4), 0]])
```
When setting source points I choose where the lane lines going on an image with a straight road. The destination points set a rectange, in which I expect these lines to be shown vertical.

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 577, 460      | 320, 0        |
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 703, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

### Identifying and Fitting Lane Lines

To identify lane lines, firstly I take a histogram along all the columns in the lower half of the image. In this histogram, there are two peaks which are the most likely x-positions where lane lines can be found.

For histogram, I created a function: called `hist` which takes a binary image as input and returns its vertical sum of image pixels as histogram. It is found in the 14th code cell of the IPython notebook.

Then I used sliding windows starting from bottom x-position to move upward in the image (further along the road) to determine where the lane line go.
To get each lane line, I also divide the image at mid vertical line: left and right images.

In function `find_lane_pixels` which appears in the 15th code cell of the IPython notebook. I use below parameters to detect lane points.

```python
# HYPERPARAMETERS
# Choose the number of sliding windows
nwindows = 9
# Set the width of the windows +/- margin
margin = 100
# Set minimum number of pixels found to recenter window
minpix = 50
```
Then I fit a 2nd order polynomial to the points (for each line - left and right) found using `polyfit` in function `fit_polynomial` which is in the 16th code cell.

I tested these functions on test images and showed the sliding windows on image:

![alt text][image5]

For video images, I don't need to do a blind search again, therefore I used previous lane line positions in functions `fit_poly` and `search_around_poly`. These functions can be found at 18th and 19th code cells of the IPython notebook respectively. I used only one parameter for this second fitting: margin.

I used  
```python
# HYPERPARAMETER
    margin = 100
```
Here is an example of searching around poly below.

![alt text][image6]

### Calculating radius of curvature of the lane and the position of the vehicle with respect to center.

I fitted lane lines to 2nd order polynomial. The function of second order polynomial is:

$$f(y)=Ay^2+By+C$$

Using the polynomial constants the radius of curvature can be calculated using the formula below. There is a proof [in this link.](https://www.intmath.com/applications-differentiation/8-radius-curvature.php)

$$R_{curve}={(1+(2Ay+B)^2)^3/2 \over |2a|}$$

Firstly I've calculated radius of curvature in terms of pixel values. I used a function named `measure_curvature_pixels` at 20th code cell of the IPython notebook.  


U.S. regulations that require a minimum lane width of 12 feet or 3.7 meters, and the dashed lane lines are 10 feet or 3 meters long each.

Therefore I use following constants to convert radius of curvature values from pixel to meter values in function `measure_curvature_real` in IPython notebook code cell with number 22.
```python
# Define conversions in x and y from pixels space to meters
ym_per_pix = 30/720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meters per pixel in x dimension
```
For calculating the position of the vehicle with respect to center, I assumed the camera is mounted at the center of the car, such that the lane center is the midpoint at the bottom of the image between the two lines I've detected. The offset of the lane center from the center of the image (converted from pixels to meters) is the distance from the center of the lane.

The function named `measure_midpoint_lanes` in IPython notebook code cell 24 calculates the position of vehicle with respect to center.

### Output visual display

Finally I used again perspective transform to get the actual lines onto the original image. However, this time I use inverse of the perspective matrix that I calculated in 12th code cell.

The function `unwarp_image` in 26.th code cell in IPython notebook draws the lane onto the warped blank image. Then warps the blank back to original image. Finally combines the result with the original image.

I, then used `cv2.putText` in 28.th code cell in IPython notebook to write mean radius and position of vehicle on image

An example of the result is below.

![alt text][image6]

---

### Pipeline (video)

I've applied all of the methods above to a test video in 34th code cell of the IPython notebook.

Here's a [link to my video result](./output_videos/project_video.mp4)
![alt text][video1]
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
