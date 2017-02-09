# Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
## What is this project
This project is about using computer vision techniques to find the lane lines on the road and fit quadratic curves on them.
## My approach
First, I calibrate the camera.

Then, I use color and gradient filter to filter out the lane lines. The color channels I use this time is "HLS" and the gradient filtering technique I use is Sobel.

Then, I add gaussian blur on image to reduce the noise.

Then, I add region filtering and perspective transformation with the same trapizium parameters to gain a bird-eye view of the lane lines.

Finally, I fit the left line and right line directly using polyfit with degree 2.

## Reflection
This is my first try of this challenging task.

I think there are several facts to be improved.

First, the noises, such as vehicles, other lane lines and shawdows, is very hard to be filtered. I should choose a better filtering algorithm to get the lane lines out only.

Second, the peak finding, and the polynomial fit, are bad engineered this time. I fit the curve directly without any noise filtering algorithm like sliding windows. Noise unfiltered is one important aspect. So do the algorithm chosen to plot the pixels on fitting space is not robust. I should try other methods on those issues.

Then, I do not know how to detect the vehicle position according to lane lines.
