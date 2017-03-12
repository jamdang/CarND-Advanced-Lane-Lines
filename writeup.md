##Writeup

---

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

[image1]: ./examples/undistort_xz.png "Undistorted"
[image2]: ./examples/distort_correct_xz.png "distort correction"
[image3]: ./examples/edge_detection_xz.png "Binary Example"
[image4]: ./examples/perspective_trans_xz.png "Warp Example"
[image5]: ./examples/lane_find_fit_xz.png "Fit Visual"
[image6]: ./examples/sample_xz.png "Output"
[video1]: ./examples/project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in code cell 2 and 3 of the IPython notebook located in "./examples/P4_AdvancedLaneFinding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objPoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgPoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objPoints` and `imgPoints` to compute the camera calibration and distortion coefficients ('mtx', 'dist') using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
By applying the `cal_undistort()` function in cell 3 with the obtained camera calibration/distortion coefficients ('mtx', 'dist'), I get the distortion-corrected image of one of the test images:
![alt text][image2]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image for edge detection. `abs_sobel_thresh()` function in cell 6 implements the directional gradient; `mag_thresh()` in cell 7 implements the gradient magnitude;  `dir_threshold()` in cell 8 implements the gradient direction; `hls_select()` in cell 9 implements the color space and s channel selection.  Here's an example of my output for this step. 
![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I used `cv2.warpPerspective()` to perform the perspective transform. To obtain the transform matrix ('M'), I chose the hardcoded source points (`src`) and destination points (`dst`) in the following manner (cell 14):  

```
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 45, img_size[1]],
    [(img_size[0] / 2 + 65), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203.3, 720      | 320, 720      |
| 1111.67, 720     | 960, 720      |
| 705, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

After I obtain the warped, masked edge binary image, I then implement the `laneFind_BoxMethod()` in cell 17 to find the lane line pixels and then fit the lane lines with a 2nd order polynomial. 
The function employes the sliding window method and basically works in the following steps:

1) identify, in the warped binary image, the starting points for left and right lane lines by finding the respective peak of pixels in the left and right bottom half of the image 

2) "slide" a window, one for the left and one for the right lane line, from the starting point at the bottom, all the way to the top of the image, finding all pixels in the window (box), while adjusting the window's horizontal location based on the pixel distribution

3) fit a 2nd order polynomial using `np.polyfit()` function


![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I implemented `curveRad()` function in cell 20 for radius calculation.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `LaneFinding_Pipeline()` in code cell 29.  Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./examples/project_video_output.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In the pipeline I implemented `laneFind_BoxMethod()` (cell 17 ) and `laneFind_BaseMethod()` (cell 18) respectively. The first one is the sliding window method which is used to find/fit the lane line from scratch, and the second method is if we already find the lane lines and then we can search near the previous lane lines. I also implemented a kind of 1st order filter to filter the lane line coefficients (curvature, heading and lateral offset) to smooth the output. These methods seem to work well on the "project_video", withstanding the shadow and pavement color changes, proving that this pipeline indeed works. However, for the "challenge_video" I didn't get quite satisfying result. I tried to improve my `laneFind_BaseMethod()` and filtering/smoothing, which doesn't really improve the final result. My debugging led me to believe that I do need better method to "find lane lines from scratch", especially when there are multiple lane line candidates in one side (left or right) of the image, e.g., in the "challenge_video.mp4", the shadow on the left will be perceived as another possible lane line very close to the actual left lane line, the sliding window (or other equivalent method) should be able to choose between these two possible lane lines. If I had more time I'd modify my `laneFind_BaseMethod()` function.

