
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

[image1]: ./output_images/test_undist.jpg "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./output_images/thresholded.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image41]: ./output_images/grid_undist_warped.jpg "Row undist,warp"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/pipline.jpg "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### 1. Camera Calibration


##### 1.1 Camera Calibration

The first step in the project is camera calibrations which is done with the provided images in the camera_cal folder. Using findChessboardCorners the corners are extracted and fed into the calibrateCamera function. This function then provides us with our image matrix and the distortion coefficents. With this information we can move onto step 2.

##### 1.2 Distortion Correction

Using the image matrix, the distortion coefficents and the undistort function the images can be properly undistored. To check that the images have been properly undistored a before and after has been applied to a checker board pattern.
![alt text][image1]

###2. Color and Gradient Threshold

I used a combination of color and gradient thresholds to generate a binary image.  Here's an example of my output for this step.

![alt text][image3]

### 3. Perspective Transform

Now for this part I decided to transform before applying my thresholds on the image. (Contrary to how the helper functions are written). This was done with the getPerspectiveTransform and warpPerspective funtions.

Transform Points

| Source        | Destination | 
|:-------------:|:-----------:| 
| 220, 720      | 320,720     | 
| 1110, 720     | 920, 720    |
| 570, 470      | 320, 1      |
| 722, 470      | 920, 1      |

Visualization of Transform

![alt text][image41]

### 4. Detect Lane Lines
Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

##### Figuring out bad frames
There will be some frames where no lanes will be detected or the lanes might not make sense. We determine the bad frames if any of the following conditions are met:
No pixels were detected using the sliding window search or search around the previously detected line.
The average gap between the lanes is less than 0.7 times pr greater than 1.3 times the globally maintained moving average of the lane gap.
##### Averaging lanes
The lane for each frame is a simple average of 12 previously computed lanes. This is done in the averaged_line method in the code block below.
##### What to do if a bad frame is detected?
Perform a sliding window search again (this is done in the brute_search method in the code block below
If this still results in a bad frame then we fall back to the previous well detected frame.
##### Final Pipeline
We combine all the code described in the code block above, plus the averaging and fallback techniques described in this block. The final code is in the pipeline method.

#### 5. Computing the radius of curvature and center offset.
The radius of curvature is computed according to the formula and method described in the classroom material. Since we perform the polynomial fit in pixels and whereas the curvature has to be calculated in real world meters, we have to use a pixel to meter transformation and recompute the fit again.
The mean of the lane pixels closest to the car gives us the center of the lane. The center of the image gives us the position of the car. The difference between the 2 is the offset from the center.

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The key challenge associated with this project was time. 
1. Histogram/sliding window: the same lane detection algorithm is used for each frame.
2. Future improvements to avoid the algorithm from failing is a much cleaner bit mask extraction. When tested on the most difficult video the main problem was additional noise creating points that ruined the polynomial fit. With more time a better bit mask extraction can be applied to enhance this algorithm.