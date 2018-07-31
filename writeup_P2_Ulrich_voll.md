## Writeup P2, Advanced Lane Finding, Ulrich Voll



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

[image1]: ./output_images/undistort_chessboard.png "Undistorted"
[image2]: ./output_images/undistort_road_image.png "Road Transformed"
[image3]: ./output_images/Thresholded_vs_original_image.png "Thresholded Binary Example"
[image4]: ./output_images/Warped_vs_unwarped_straight.png "Warp Example"
[image5]: ./output_images/Lane_lines_recogized_on_test6.png "Fit Visual"
[image6]: ./output_images/Projecting_back_with_text.png "Output"
[video1]: ./output_project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I consider the rubric points individually and describe how I addressed each point in my implementation in [this .jpynb](https://github.com/uv1000/P2/blob/master/P2_SDC_Ulrich_Voll.ipynb).  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I implemented this in section 1 of my .jpynb.

I adapted 
[this .jpynb](https://github.com/udacity/CarND-Camera-Calibration/blob/master/camera_calibration.ipynb) provided in the classroom. 

The code for this step is contained in the code cells of Section 1 of my IPython notebook located in
"./P2_SDC_Ulrich_Voll.ipynb".

Here the grid `objp` is constant for all images, representing cordinates of an idealised 9x6 chessboard in three-space (z = 0). 

In a loop over a list containing all example pictures I attempt to find chessboard coordinates, using cv2.findChessboardCorners().

Whenever I succeed in finding some, the resulting `corners` to another list called `imgpoints`, and I also append `objp` to the list `objpoints`.

In case no chessboard is found by  cv2.findChessboardCorners(), I append nothing to those lists. 

I then use the resulting lists `objpoints` and `imgpoints` to compute the camera calibration and 
distortion coefficients using the `cv2.calibrateCamera()` function.  Apparently this function is capable of accepting and averaging over an entire list of pairs (opjp,corners). 

I apply this distortion  correction to one of the test images using the `cv2.undistort()` function and obtained this result: 

Distorted vs undistorted (calibration3.jpg):
![alt text][image1]

Finally I save the coefficients mtx and dist to a pickle container.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I implemented this in section 2.1 of my .jpynb.

Applying `cv2.undistort()` using the parameters/coefficients from above to test_images/test1.jpg yields:
![alt text][image2]

I chose not to write a separate wrapper-function around cv2.undistort() just for the sake of its own, in order avoid bloating the code.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I implemented this in section 2.2 of my .jpynb.

In my function thresholding, I adapted the quiz .py file developed in the classroom. 

Thresholded Image 

= Binary picture 

= Logical OR of thresholding S value of HLS transform and Sobelx gradient. 

Details including threshold values see .jpynb. 

This yields:
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I implemented this in section 2.3 of my .jpynb.

I perform the perspective transform in my self-defined function called `unwarp()`.  

The `unwarp()` function contains hand-tuned (`src`) and destination (`dst`) points. As in the respective quiz in classroom. I hand tuned the src- and dst-point by validating against the "straight" example pictures. 

I chose to hardcode them as follows:

```python
 src = np.float32(
     [[(img_size[0] / 2) - 60, img_size[1] / 2 + 100],
     [((img_size[0] / 6) - (5)), img_size[1]],
     [(img_size[0] * 5 / 6) + 45, img_size[1]],
     [(img_size[0] / 2 + 65), img_size[1] / 2 + 100]])
    dst = np.float32(
     [[(img_size[0] / 4), 0],
     [(img_size[0] / 4), img_size[1]],
     [(img_size[0] * 3 / 4), img_size[1]],
     [(img_size[0] * 3 / 4), 0]])
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I implemented this in section 2.4 of my .jpynb.

I adapted the sliding window algorithm from quizz/classrom for generating two 2nd order polynomial.  

I slightly adapted the functions find_lane_pixels() and fit_polynomial() from the quizz.

Mainly I had to adapt the height of y-values summed for the initializing histogram (lower 2/3 instad of lower 1/2, as in the classroom quiz/example). I payed to increase the parameter "margin" from 100 to 125 at some stage, but it is back to 100 in the present implementation.   

I also added some visualisation (paint lane found green), in fit_polynomial.

The resulting algorithm in Section 2.4 of the .jpynb now yields satisfactory results for all test images provided.

E.g. for image test6 I got:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I implemented this in section 2.5 of the .jpynb.

My function measure_curvature_and_offset_real(ploty, left_fit,right_fit) consumes a vector of y-values and polynomial coefficients (all provided by function fit_polynomial() in the previous section 2.4) 

My helper function convert_to_metres() helps transforming everything from pixel scale to scale in metres, similarily as in the respective quiz in the classroom.

y_eval (in metres!) is the maximum y-value, corresponding to the bottom of the image.

Curvature is computed using the formulae given in the classroom:

    
    A=left_fit_cr[0]
    B=left_fit_cr[1]
    left_curverad = 1/(2*A)* (1+(2*A*y_eval+B)**2)**(3/2)
    

Offest is computed by simply forming the arithmetic mean between the closest right lane-point and the closest left lane-point. It is computed relative to the middle point in x (hard-coded by hand). 

     
    leftx_bot = left_fit_cr[0]*y_eval**2 + left_fit_cr[1]*y_eval + left_fit_cr[2]
    rightx_bot = right_fit_cr[0]*y_eval**2 + right_fit_cr[1]*y_eval + right_fit_cr[2]
    avx_bot=1/2*(leftx_bot +rightx_bot)
    offset_x_m= 1280/2*xm_per_pix -avx_bot
    
    

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this in section 2.6 of the .jpynb.

My function project_back(img,left_curverad, right_curverad,offset_x_m) projects the output image of pipeline step 2.4 (Fit lines with polynomial.) back onto the road, using a function warp_again() which is the inverse of the above warp(). 

My new helper-function warp_again()  is essentially interchanging src-points and dst-points when calling cv2.getPerspectiveTransform()) relative to warp() from pipline step 2.3 (Apply perspective transform.) 

Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
