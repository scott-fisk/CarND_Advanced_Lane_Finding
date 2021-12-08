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

[image1]: ./output_images/cal_undistort.jpg "Undistorted"
[image2]: ./output_images/undistorted_road.jpg "Road Transformed"
[image3]: ./output_images/combined_image.jpg "Binary Example"
[image4]: ./output_images/transformed_image.jpg "Warp Example"
[image5]: ./output_images/histogram.jpg "Histogram"
[image6]: ./output_images/polynomial_image.jpg "Fit a polynomial"
[image7]: ./output_images/search_poly.jpg "Search around poly"
[image8]: ./output_images/final_image.jpg "Final result"
[video1]: ./output_videos/project_video_output.mp4 "Video"

---

### Writeup / README


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./pipeline.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Distortion-corrected image.

![alt text][image2]

The distorted image is undistorted using function cal_undistort(img, objpoints, imgpoints).

#### 2. Binary image .

I used a combination of color and gradient thresholds to generate a binary image using funtion combined_thresholds(img, sx_thresh=(20, 100), s_thresh=(170, 255)). The image is coverted to HLS color space and the S channel is separated. Sobel X, threshold X gradient, and threshold color channel are applied to the input image and then a image is returned. 

Here's an example of my output for this step.

![alt text][image3]

#### 3. Perspective transform.

The code for my perspective transform includes a function called transform(src, dst, undist_image). cv2.warpPerspective() is used to warp the image to a top-down view within this function. 

The thresholded binary image (binary_image) is used for the perspective transform.

The `transform()` function takes as inputs an image (`undist_image`), as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points in the following manner:

src = np.float32([[580,450],[690,450],[1050,700],[270,700]])

dst = np.float32([[220,0],[1080,0],[800, 720],[330,720]])

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 450      | 220, 0        | 
| 690, 450      | 1080, 0       |
| 1050, 700     | 800, 720      |
| 270, 700      | 330, 720      |


![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The transformed image (top_down) is then used in a histogram to determine where the pixels are located.

![alt text][image5]

The two peaks indicate the left lane and right lane. 

find_lane_pixels(top_down) uses the histogram to determine the starting point for the left and right lanes. This is then used in another function fit_polynomial(top_down) along with second order polynomial to draw boxes onto the image. The boxes have specific values:

Choose the number of sliding windows
nwindows = 9

Set the width of the windows +/- margin
margin = 90

Set minimum number of pixels found to recenter window
minpix = 45

![alt text][image6]

Using values from this polynomial another function is created search_around_poly. The sliding windows are no longer seen and a polygon is generated to show the search window area. 

![alt text][image7]


#### 5. Lane curvature and vehicle position.

measure_curvature_real(left_fit, right_fit) is used to calculate the curvature of polynomial functions in meters. 

measure_car_pos(binary image, left_fit, right_fit) is used to calculate the cars positions in the image. 

#### 6. Final result

The function `pipeline()` contains all the various functions described above. Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline (video)

#### 1. Project video

Here's a [link to my video result](./output_video/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I ran into issues implementing the pipeline. Images that were used previously were overwriting the image I was trying to use in the pipeline. Processing the video was also problematic. It was using the test image being processed in the pipeline rather than the video. All these bugs were eventually worked out by making sure the functions didn't carry over information from any previous steps. 

Most of the other issues were minor tweaks to the parameters for example src and dst points. 
