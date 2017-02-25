##Writeup Template


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
[image2]: ./test_images/test1.png "Road Transformed"
[image3]: ./examples/binary_combo_example.png "Binary Example"
[image4]: ./examples/warped_straight_lines.png "Warp Example"
[image5]: ./examples/color_fit_lines.png "Fit Visual"
[image6]: ./examples/example_output.png "Output"
[video1]: ./trial_203.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You are reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "Stage 1: Calibration".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in my code cell "Stage 2: Color and Gradient")  Here is an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective()`, which appears in the stage 3 of the IPython notebook.  The `perspective()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
# Note: x1, x2 and h are the parameters to be defined
x1=params_p[0]
x2=params_p[1]
h=params_p[2]
m1=360.0/(640-x1)
m2=360.0/(640-x2)
dx1=h/m1
dx2=h/m2

src=np.float32([
[x1,720],
[x1+dx1,720-h],
[x2+dx2,720-h],
[x2,720]
])
dst=np.float32([
[x1,720],
[x1,0],
[x2,0],
[x2,720]
])
```
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in "Stage 4: Detect curverad and vehicle position"

The function "line_finding" takes binary warped image as input.

If this is the first frame, the sliding windows technique is used to find the lane lines. First, get the starter position of the sliding by getting the peak of the bottom half of the image. Then slide up step by step by 1/8 the image, find the peak of each sliding and keep on. Then region filter the image according to the peaks and that accurately get the lane lines out and filter other noises.

With the detection of the last frame, the finding of next frames are much easier. Just find the lane lines within the regions of the lane lines found frame as the change between frames is very small. 

Then I add the smoothing technique in order to filter noise between frames. The smoothing method I use this time is learned from Vivek Yadav in his repo "vxy10/p5-VehicleDetection-Unet". If the difference of quadratic factors betwwen two frames is larger than a threshold, then use the last fit. Else, the result is 5% of the current fit and 95% of the last fit.


####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in "Stage 5: Warping Back" . Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here is a [link to my video result](./trial_203.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I will talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

First, if the road that the car is driving has no lane lines, too much voices effect the detection result(like in harder challenge video), or the road sign is placed between the lane lines, my pipeline may incorrectly recognize the road sign as lane line and that would cause a bad fit. To make it robust enough to execute on a real car, I should add a verification function that returns whether there is a lane line. If the verification function returns False, then the pipeline is paused and the car system would not believe the result of the lane line detection. 
