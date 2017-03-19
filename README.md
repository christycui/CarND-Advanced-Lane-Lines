## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


In this project, I aim to detect the lane area in a video of car driving on highway. The original video is 'project video.mp4'. The result video with annotated lane detection is 'lanes.mp4'.

[//]: # (Image References)

[image1]: ./output_images/chessboard_undistort.jpg "Undistorted"
[image2]: ./output_images/undistort.jpg "Road Transformed"
[image3]: ./output_images/combined_binary.jpg "Binary Example"
[image4]: ./output_images/perspective_transform.jpg "Warp Example"
[image5]: ./output_images/detect_lanes.jpg "Fit Visual"
[image6]: ./output_images/lanes_painted.jpg "Output"
[video1]: ./lanes.mp4 "Video"

The Project
---

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
![alt text][image1]
* Apply a distortion correction to raw images.
![alt text][image2]
* Use color transforms, gradients, etc., to create a thresholded binary image.
![alt text][image3]
* Apply a perspective transform to rectify binary image ("birds-eye view").
![alt text][image4]
* Detect lane pixels and fit to find the lane boundary.
![alt text][image5]
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.
![alt text][image6]
