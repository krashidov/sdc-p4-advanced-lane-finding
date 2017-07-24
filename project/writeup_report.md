
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
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text](https://raw.githubusercontent.com/krashidov/sdc-p4-advanced-lane-finding/master/project/undistorted.jpg)

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text](./sdc-p4-advanced-lane-finding/project/undistorted.jpg)

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps are in cell 3 marked Color and Gradient Thresholding).  Here's an example of my output for this step. 

![alt text](https://raw.githubusercontent.com/krashidov/sdc-p4-advanced-lane-finding/master/project/thresholding.jpg])

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
    #Source Points
    width = self.image_width(image)
    height = self.image_height(image)
    bottom_width = 0.76
    middle_width = 0.08
    height_pct = 0.62
    bottom_trim = 0.935

    top_left = [width * (0.5 - middle_width/2), (height * height_pct)]
    top_right = [width * (0.5 + middle_width/2),(height * height_pct)]
    bottom_right = [(width *(0.5 + bottom_width/2)), (height * bottom_trim)]
    bottom_left = [(width * (0.5 - bottom_width/2)), (height * bottom_trim)]

    #Destination Points
    #A fifth of the width of the image
    offset = width * 0.2

    #top left, top right, bottom right, bottom left
    destination = np.float32([[offset, 0], [width-offset, 0], [width-offset, height], [offset, height]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 588.8, 446.4  | 256, 0        | 
| 691.2, 446.4  | 1024, 0       |
| 1126, 673     | 1024, 720     |
| 153, 673      | 256, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text](https://raw.githubusercontent.com/krashidov/sdc-p4-advanced-lane-finding/master/project/warped.jpg)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the histogram method to idenitfy lane lines
![alt text][https://raw.githubusercontent.com/krashidov/sdc-p4-advanced-lane-finding/master/project/anefinding.jpg]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the process_image method of the tracker class, under the comment marked "Calculate the new radii of curvature"
This code is based on the video walkthrough provided by udacity. First we need a way to convert from pixels to meters. I define that as 30 meters per pixel in the Y direction and 3.7 meters per pixel in X direction. To calculate the radius of curvature you essentially have to fit a circle around a sloped line. This can be useful for things like anamoly detection and later on, controlling the vehicle. The radius of this circle is the reciprical of the derivative of theta with respect to x. Where theta is the angle between a tangent line and the instantaneous point of where we are trying to calculate the radius of the curve. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the `process_image` method of the tracker class under the comment marked " # Warp the blank back to original image space using inverse perspective matrix (Minv)"

![alt text][https://raw.githubusercontent.com/krashidov/sdc-p4-advanced-lane-finding/master/project/output0.jpg]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./sdc-p4-advanced-lane-finding/output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My pipeline will fail when the tracker gets confused by very dark shadows in the median, or when there are odd markings within the lane lines. I think this is fairly easy to mitigate by telling the algorithim that a lane line should have sane dimensions. Improved thresholding and masking could also mitigate this problem.
