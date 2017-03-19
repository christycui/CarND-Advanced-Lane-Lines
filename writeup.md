##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./output_images/chessboard_undistort.jpg "Undistorted"
[image2]: ./output_images/undistort.jpg "Road Transformed"
[image3]: ./output_images/combined_binary.jpg "Binary Example"
[image4]: ./output_images/perspective_transform.jpg "Warp Example"
[image5]: ./output_images/detect_lanes.jpg "Fit Visual"
[image6]: ./output_images/lanes_painted.jpg "Output"
[video1]: ./lanes.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the Camera Caliberation section of the IPython notebook located in "./Advanced Lane Finding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
I first corrected the camera distortion in the test image using the distortion coefficient "dist" returned in the previous camera caliberation step. I applied 'cv2.undistort()' to obtain the undistorted image:
![alt text][image2]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image. In particular, I applied x and y gradient, set threshold for magnitude and direction of gradient and applied color threshold on the s-channel. 

Here's an example of my output for this step:

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in code cell 418 of the IPython notebook.  The `warp()` function takes as inputs an image (`img`).  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32(
      [[567,465],
      [567+132,465],
      [1080,672],
      [313,672]])
dst = np.float32(
      [[100,0],
      [1000,0],
      [1000,720],
      [100,720]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 567, 465      | 100, 0        | 
| 699, 465      | 1000, 0      |
| 1080, 562     | 1000, 720      |
| 313, 672      | 100, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
There are two approach in finding the pixels.
The first one is in 'detect_lane_pixels' method (code cell 556). to break the warped image into 9 windows and I created histograms to search for peak of pixel values in each window. After finding the lane pixels I fited a 2nd order polynomial kinda like this:

![alt text][image5]

The second approach (code cell 596) is used when lanes were found in the previous frame. It searches around the lanes found in previous frame (I set a margin of 100). After finding the lane pixels I fited a 2nd order polynomial like in the first approach.

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

This was accomplished in code cell 635, 'calculate_curvature()'. First I set meter per pixel in x and y dimensions. Then using the left_fit and right_fit coefficients and assuming the bottom of the image to be the radius of the curvature, I was able to find out the real-world radius of the curve using the curvature formula as presented in the lesson.

To find out the position of the vehicle, I used the left and right lane line fits to find out the middle of the lane at the bottom of the image. Assuming the camera is mounted in the center, the center of the image should be the middle of the road. So, the difference between the center of the image and the middle of the detected lanes is the offset of the vehicle.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 635 in 'draw_lanes()'. I used inputs left_fit and right_fit to calculate the x values of lane lines. Then recast the left and right lane points onto the undistorted image. I also used 'cv2.fillPoly()' to color fill the lane area. I also wrote the text of the measurements on to the image. Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

To track lane lines throughout the video, I created Lane class to keep track of both left and right lanes. I start by initialize the left and right lane objects. I undistort, apply gradient and warp the image, then when there is no previous frame, I apply window search to find lanes. If there is a previous frame, I search around the previously found lanes.

Then I calculate the curvations given the line fit. If any of the curves is smaller than 100 meters, it means it is a bad dectection and throw away. If the two lanes are somewhat parallel (difference smaller than 600 when curved), it means the detection of lanes is valid and save the fits to current_fit attribute of the lane objects. Lastly, I draw the lanes back down to the original image.

Here is the output of the project video:
![alt text][video1] (/lanes.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

One problem I faced is that, no matter what parameters I choose in perspective transform, individual frame lane detection does not perform consistently, meaning it works great for some frames but not for others. This is why smoothing is so important for video performance. 

I implemented smoothing by judging the returned curvations - does the magnitude of curvation makes sense? If it's less than 100 meters, it probably means something was not detected correctly, and that this frame should rely on previous tested detections, rather than the result from this frame. I kept 5 most recent valid fits and that worked well for most of the frames in the project video.

Another way I can see smoothing being implemented is tracking the lane pixels detected instead of the curvations. This approach might be closer to the core of the detection algorithm. The idea is that when we keep the lane pixels detected and compare those to the ones in the previous frame, and see how far apart or close to each other they are. If they are fairly similar to each other, it is a good detection because one frame is not going to change much from the previous one. If the detected pixels from this frame vary greatly from the last one, something probably went wrong. I see this pixel approach as a possible improvement for my current pipeline.

