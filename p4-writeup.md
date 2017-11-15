
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
Image
[//]: # ( References)

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

### Camera Calibration

#### 1. Computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first 5 code cells of the IPython notebook located in "find_lane.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. An example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text](https://github.com/solo2002/CarND-Advanced-Lane-Lines/blob/master/output_images/undisort.png?raw=true)

#### 2. Examples of color transforms, gradients or other methods to create a thresholded binary image. 

I tried several combinations of color and gradient thresholds to generate a binary image (please see code sells from 6 to 13 in "find_lane.ipynb").  Here's an example of combinaing red channel and S channel.

![alt text](https://github.com/solo2002/CarND-Advanced-Lane-Lines/blob/master/output_images/r-s-channel.png?raw=true)

Here is the combination of sobel_x, S channel, magnitude of the gradient, and direction of the gradient.

![alt text](https://github.com/solo2002/CarND-Advanced-Lane-Lines/blob/master/output_images/combined.png?raw=true)

#### 3. Perspective transform.

The code for my perspective transform includes a function called `warp()`, which appears in code cell 18 in the file "find_lane.ipynb".  The `warp()` function takes as inputs an image (`img`).  I chose the hardcode the source and destination points in the following manner, and this resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 490, 480      | 0, 0        | 
| 810, 480      | 1280, 0      |
| 1250, 720     | 1250, 720      |
| 0, 720      | 40, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text](https://github.com/solo2002/CarND-Advanced-Lane-Lines/blob/master/output_images/warp.png?raw=true)

#### 4. Identified lane-line pixels and fit their positions with a polynomial?

In code cell 15, I fit my lane lines with a 2nd order polynomial kinda like this:

First, calculating the histogram of the image, and the hightest 2 peaks correspond to lane lines. Then siliding-window methond is using to find lane lines. Finally, these two steps approaches are implemented for each frame of images. Here are the example images of these: 

![Histogram](https://github.com/solo2002/CarND-Advanced-Lane-Lines/blob/master/output_images/histogram.png?raw=true)

![Fit-line](https://github.com/solo2002/CarND-Advanced-Lane-Lines/blob/master/output_images/fit-poly-line.png?raw=true)


#### 5. Calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code cell 18 in the file "find_lane.ipynb". Generally, the curvature for left and right lane lines are determined by their radius, respectively. Then the relation between pixes and meters is calculated based on the US high way specification: 700 pixes vs 3.7 meters.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![Lane-line](https://github.com/solo2002/CarND-Advanced-Lane-Lines/blob/master/output_images/find-line.png?raw=true)

---

### Pipeline (video)

#### 1. Provide a link to your final video output. 

Here's a [link to my video result](https://youtu.be/49TTZed7nuA)

---

### Discussion

#### 1. Implementation 

Previouly, I used a queue structure to save lane lines curvature from a certain number frames of image, and remove the first one as well as insert a new one when there is a significant difference. The limitation of this implementation is that it is a kind of delayed to reflect the change. Therefore, it does not work well when the road curve is dramatially changed, just like the challenge video. Now it was improved as follows. 

The L-channel of LUV is used to detect white lane, and the B-channel of LAB is used to recoginize yellow lane. 

Then I further improved the algorithm by doing following:

##### a. If lines are not detected, sliding window search is used to find the line.

##### b. If lines are not detected, only around region is searched (here +/- 25 pixel is used).

##### c. Averge lines based on last 10 frames, so lines would change more smoothly  

#### 2. Limitation 

Most of parameters used in this project is hard-coded. For instance, the threshold values, the region of image, etc. Therefore, if given another different type of video, the pipeline may fail. In addition, the accuracy could be reduced when calculating radius of curvature and offset.

Moreover, the process of tuning of parameters, including the threshold values, and combinations of image processing filters, is very time consuming.    
