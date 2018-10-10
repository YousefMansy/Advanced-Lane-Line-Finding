## **Advanced Lane Finding Project**

The goal of this project is to write a software pipeline to identify the lane boundaries in a video from a front-facing camera on a car through the following steps:

* Compute the camera calibration matrix and distortion coefficients using chessboard images.
* Apply distortion correction to frames.
* Apply perspective transformation to get a bird's eye view of the lane.
* Use color thresholding to create a binary image of the lane lines.
* Detect lane pixels and fit a polynomial to find the lane boundaries.
* Calculate the curvature of the lane and vehicle position with respect to center.
* Project the detected lane boundaries back onto the original image and display the curvature and position from center values.

[//]: # (Image References)

[image1]: ./output_images/find_corners_output/calibration11.png "Corners"
[image2]: ./figures/calibration1_distortion_fig.png "calibrated_chessboard"
[image3]: ./figures/straight_lines2_distortion_fig.png "calibrated_road"
[image4]: ./output_images/image_w_trapezoid/straight_lines2_w_trapezoid.png "trapezoid"
[image5]: ./figures/straight_lines2_perspective_fig.png "trapezoid"
[image6]: ./figures/colorspaces/original.png "colorspaces"
[image7]: ./figures/colorspaces/HLS_L.png "colorspaces"
[image8]: ./figures/colorspaces/HLS_S.png "colorspaces"
[image9]: ./figures/colorspaces/HSV_V.png "colorspaces"
[image10]: ./figures/colorspaces/LAB_B.png "colorspaces"
[image11]: ./output_images/thresholding_binary_output/straight_lines2.png "binary_thresholding"
[image12]: ./figures/straight_lines2_sliding_fig.png "sliding window"
[image13]: ./figures/straight_lines2_lane_projection_fig.png "lane projection"

### Camera Calibration

I start by calculating the object and image points for a set of images of a 9x6 chessboard taken with the same camera as the one used in the car.

The `imgpoints` represent the (x, y, z) coordinates of the chessboard corners in the photos and the `objpoints` represent the ideal chessboard positions. Only image points from photos were all corners of the chessboard were detected successfully are added to `imgpoints`, and a copy of the same object points is added to `objpoints` for each successful detection as well.

Then camera calibration and distortion coefficients `mtx, dist, rvecs, tvecs` are calculated using all the image and object points generated.

The output is saved into the `find_corners_output` directory.

The code for this step is contained in the functions titled `find_img_and_obj_points` and `calibrate_camera` in the IPython notebook located in `./P2.ipynb`

An example of a chessboard image with successful detection of corners is shown below:

![alt text][image1]

After calculating the calibration data, I test it by using it to calibrate all the chessboard images and observe the results.

An example of an undistorted chessboard image is shown below:

![alt text][image2]

The calibration results seem okay.

Then I use the same data to calibrate the test images of the road and save the output to the `distortion_correction_output` directory

The code can be found in the `correct_for_distortion_dir` function in the IPython notebook located in `./P2.ipynb`.

An example of an undistorted road image is shown below:

![alt text][image3]

### Pipeline

#### 1. Perspective Transformation

I used a simple lane detection pipeline on one of the test images with straight line lanes to extract a trapezoid representing the lanes to use it later for perspective transformations.

Note that this trapezoid is generated once then used for all transformations later. The reason we don't need to re-generate it for each frame is that the vertical angle of the camera is fixed in all photos so the same trapezoid should be valid for all of them.

The code can be found in the `find_trapezoid` function in the IPython notebook located in `./P2.ipynb`.

The output is saved to the `image_w_trapezoid` directory

![alt text][image4]

Now that I have the trapezoid shape, I transform this shape into a rectangle filling the frame vertically to get a Bird's Eye Perspective Transformation of the road. This makes it easier to fit a polynomial to the lanes.

The code can be found in the `trapezoid_birds_eye_view_transform_dir` function in the IPython notebook located in `./P2.ipynb`.

The output is saved to the 'perspective_transformation_output' directory.

![alt text][image5]

#### 2. Binary Thresholding

In the next step, I observe four different colorspace channels and try to determine which ones best suit our needs for extracting the white and yellow lane lines.

I tested the HLS L channel, HLS S channel, LAB B channel, and finally HSV V channel.

![alt text][image6]
![alt text][image7]
![alt text][image8]
![alt text][image9]
![alt text][image10]

By observing the results above, I noticed that the HLS L channel is good at separating the white lanes, and the LAB B channel is good at separating the yellow lanes.

After some trial and error, I combine both channels to generate a binary image using `220, 255` thresholds for the L channel, and `160, 170` thresholds for the B channel.

The code can be found in the `generate_thresholded_binary_dir` function in the IPython notebook located in `./P2.ipynb`.

The Binary Thresholding output is saved to the `thresholding_binary_output` directory.

![alt text][image11]


#### 3. Lane Line Fitting

Next, I used a technique called the Sliding Window to extract a polynomial for each lane line. The idea is to iterate a fixed size window over the image, starting from the bottom of the lane and going up with a fixed step size, each time shifting its position left or right to recenter it above the line.

Initially, we calculate a histogram for the binary image, then look for the highest values in the second and third quadrants to use them as the starting points for the windows.

After some trial and error, I settled for using 8 window steps vertically.

The code can be found in the `find_lanes_dir` function in the IPython notebook located in `./P2.ipynb`.

The output is saved into the `sliding_window_output` directory.

![alt text][image12]

#### 4. Lane Projection and calculation of curvature and drift from center

Finally, after we've successfully extracted and plotted the two polynomials representing the two lanes, we project those lines again onto the original image by performing the Inverse Perspective Transformation to the transformation we used earlier.

We also use the polynomials to calculate the radius of curvature in meters (since we know what the value of the real distance between the lanes should be)

Finally, we calculate how far off from the center of the two lanes the car is in meters.

The output is saved into the `lane_projection_output` directory.

![alt text][image13]

---

### Video

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

I believe the current pipeline performs reasonably well on the project video linked above. However, upon testing on more challenging videos [such as this one](./challenge_video_output.mp4), the results are significantly less reliable.

I believe this is due to the fact that in this second video the lighting conditions are more challenging, and since my pipeline relies only on color thresholding, it fails in situations were the color of the lane lines and the road aren't distinguishable enough. I could try further enhancing the pipeline in the future to also make use of gradient thresholding to enhance the results.

Furthermore, I have noticed that in this video my current pipeline performs a somewhat correct detection of the road lane lines every second or so, then drifts into another false detection, then back into a correct one. I believe that if I add a `search from previous` functionality to the sliding window algorithm it would help enhance the results in this video.  
