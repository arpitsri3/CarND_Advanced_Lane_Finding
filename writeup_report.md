## Writeup Report


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

[image1]: ./Visualizations/Visualization1.png 
[image2]: ./Visualizations/Visualization2.png 
[image3]: ./Visualizations/Visualization3.png 
[image4]: ./Visualizations/Visualization4.png 
[image5]: ./Visualizations/Visualization5.png 
[image6]: ./Visualizations/Visualization6.png 
[image7]: ./Visualizations/Visualization7.png 
[image8]: ./Visualizations/Visualization8.png 


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it! The code for this project is contained in the jupyter notebook by the name of 'code.ipynb' in the project folder.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for computing the camera matrix and distortion coefficients is contained in the jupyter notebook (Cells 4, 5 and 6) by the name of 'code.ipynb' in the project folder.

Here's a visualization of the images to be used for calibration:
![alt text][image2]

I have used the same implementation for detecting and storing the 'objpoints' (coordinates of the real 3D world) and the 'imgpoints' (coordinates in a 2D image) that has been used in the classroom with minor modifications to store the result of drawing chessboard corners on the calibration images for visualizations later. As discussed in the writeup-template , the chessboard is assumed to be fixed on the (x,y) plane at z=0. For each of the calibration images where the chessboard corners are detected , they are appended to the imagepoints array. The 'Object Points' being just the real world coordinates are a copy of the chessboard corners in real world.
 
Here's a visualization of the images after drawing chessboard corners:
![alt text][image3]
 
The `objpoints` and `imgpoints` are used to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image4]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Here's a visualization of the images to be used for testing:
![alt text][image1]

Here's a visualization of the images after undistortion(Column 1 - Original ; Column 2- Undistorted ; Column 3 - Warped): 
![alt text][image5]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

(Note- I have done Perspective transform before the image processing)

I really enjoyed this part of the project as I felt that this had the maximum room to play around with and i got some really weird ( :-D ) results at times but ultimately I used a combination of the following:

1. A gradient magnitude threshold.
2. Converted image to HSL, extracted the Hue channel and applied a hue range from 39 to 69 which is rougly close to yellow range.
3. Converted image to HSL, extracted the Lightness channel and applied a range from 95 to 100 for approximately capturing the white/light      portions.
4. Combined the hue channel threshold with the light channel threshold.
5. Converted image to HSL, extracted the Saturation channel and applied a threshold range from 100 to 255 for detecting regions in image      with high saturation.
6. Extracted the R channel from the image and applied a threshold range from 150, 255 as it helped in smoothing out the noise.

Following is the procedure for combining:

1. I combined the H and L binaries to obtain a result with the outline of the lane lines.
2. I combined the above once with S channel binary and once with red channel binary using 'or'.
3. I combined the two separate results from above using an 'and' operator.
4. Finally, I combined the result from above with the result of gradient magnitude binary.

The code cells for the above implementation may be found from 7-14 and 17. I tried using direction binary, sobelx and sobely while working on the image processing but my results using the above approach turned out to be more robust under varying lighting conditions. Here's an explanation or an intuition behind the above steps:

1. The hue threshold I have decided based on the values I have taken from this (very useful) site - http://colorizer.org/ . I have tried to    capture the hue values for yellow as the left lane is yellow.
2. After that I use the light channel to capture the right lane lines which are mostly white , again using values based on playing around on the site i mentioned above.
3. When the above two were combined a decent detection of right and left lines was obtained with a fair bit of noise (due to areas with comparable lightness to the white lanes)
4. On combining the above with the s_channel and the r_channel the lane detection got even better with reinforcement along the lane lines.
5. When I combined the above result together the best of both helped in lane detection over changing light condition.
6. Ultimately, when I combined the result in 5 with the gradient magnitude threshold result the noise was effectively filtered out with bright lane lines.

An additional note is that, before applying all of the above I used some of the concepts we learned in Project 1 -
1. Gaussian Blur with Kernel Size of 9
2. Canny with threshold 60, 240. 
One observation is that I have kept these ranges same as what I used in the original project. Applying these two helped in smoothing out the noise somewhat during the starting.

My key takeaway from this exercise is that I small detail which might be captured by one channel but not by other, and which may seem trivial at first can be really helpful for correct lane detection, eventually, if utilized properly.

Here's a visualization of the processing(Column 1 - Original ; Column 2- Warped Column 3 - Warped and processed )

![alt text][image6]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I applied Perspective transform before processing the image as the transformed image is more focused and detection of lane lines is more accurate on it without the presence of all the data in the image which is not relevent to our problem.

The code for my perspective transform includes a function called `corners_unwarp()`, which appears in cell 15 .  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points which we had earlier dumped as pickle from the camera calibration. .  I have simply hardcoded the source and destination points by a lot of hit and trial with the general intiution that "specifying points which are lower in the image and a bit farther away from each other". I went through the udacity forums and found this really helpful post - https://discussions.udacity.com/t/perspective-transform/235255/10 . I have saved the Minv into an array so as to warp back the detected lane lines onto the original image.

Ultimately I used following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 570,470      | 320, 0        | 
| 720,470      | 980,0      |
| 980,680     | 980,720      |
| 320,680      | 320,720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

Here's a visualization (Column 1 - Original ; Column 2- Undistorted ; Column 3 - Warped):

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the code discussed in the classroom in lessons 33 to 36 under to identify the lane lines and fit them with a polynomial. The code  is present in the cell 19.

Here's my intuition on how the code works:
1. We calculate the histogram along all the columns in the lower half of the warped and processed images.
2. We determine the midpoint and calculate the histogram peaks (using argmax function) on either side of the midpoint.
3. If our image processing is good enough, hopefully, the histogram peaks would be detected at the left and right lanes in the image.
4. We identify the non zero pixels for both x and y and save them.
5. After setting a sliding window with a range of 9 , we try to create windows with a rectangular shape and slide them upwards starting from the histogram peak. We do this by taking the mean of the non-zero pixels (by checking in the data we collected in step 4) in that box. 
6. Thus, we generate the left and right fit values to eventually plot them on the image.

Here's a visualization of applying it to the warped image and unwarping (Column 1 - Original ; Column 2- Lanes; Column 3 - Unwarped):

![alt text][image7]

Here's a visualization of just the unwarped lane lines:

![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is calculated using the formula given in the classroom and the code implementation for it given in the classroom. The same can be found in the code implemented in cell 19. The Vehicle position is calculatede by averaging the min of left points and max of right points and subtracting this mean from the image width by 2 (which is rougly the center of lane) by intuition.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

As discussed in section 4 , Here's a visualization of applying result and unwarping (Column 1 - Original ; Column 2- Lanes; Column 3 - Unwarped):

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

I tried the same pipeline on the test video - Once with the position of vehicle displayed (./test_videos_output/project_video_output1.mp4) and one without the vehicle position but both left and right curvatures (./test_videos_output/project_video_output.mp4). I was slightly surprised by the slight difference in the results at both the times. 
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I think the major problem which I faced in the course of working on this project were:
1. Getting a correct and robust (considering if there are bumps, elevation changes, etc on the road) perspective transform
2. Implementing an image processing technique which would work in varying brightness, contrast and lighting.

Considering the first point, I feel that I will work on applying a better perspective transform which can remove the effect in the first few seconds of the output video where the shape of the green area of selection is a bit crooked towards the top right.

Secondly, for the second point, I felt that the pipeline works reasonably ok on the challenge video but falls apart (terribly :/ ) in the harder video. I personally feel that I need to do a bit more research on more techniques that may be used to tackle the conditions (considerably sharper turns, more real world problems)in the harder challenge video.

Also, since I have put so many visualizations at every cell , when i began creating the pipepline for the actual video, it became kind of not easy to separate some repetitive code into functions (as I am saving the result for all test images at each step and visualizing it) , so I have used the same code (specifically for the sliding window) again in the pipeline. I also wanted to use the averaging part and just see how the result behaves which I am planning to try when I try on the hardest challenge video. Currently, I am using a sliding window for each frame which gives decent results but isn't an example of good coding. I'd like to improve on this when I try the hardest challenge video (Anyway, I already feel that it might be a necessity in the challenging conditions for that recording :) )

All that said, I really liked playing around with the thresholds for processing the image; it was interesting but also a bit tedious (I'd just feel dishonest if i didn't accept that :P) 
