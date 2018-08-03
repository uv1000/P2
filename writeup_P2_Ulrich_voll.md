## Writeup P2, Advanced Lane Finding, Ulrich Voll



The goals / steps of this project are the following:

1 Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.

2.1 Apply a distortion correction to raw images.

2.2 Use color transforms, gradients, etc., to create a thresholded binary image.

2.3 Apply a perspective transform to rectify binary image ("birds-eye view").

2.4 Detect lane pixels and fit to find the lane boundary.

2.5 Determine the curvature of the lane and vehicle position with respect to center.

2.6 Warp the detected lane boundaries back onto the original image.

3 Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


[//]: # (Image References)

[image1]: ./output_images/undistort_chessboard.png "Undistorted"
[image2]: ./output_images/undistort_road_image.png "Road Transformed"
[image3]: ./output_images/Thresholded_vs_original_image.png "Thresholded Binary Example"
[image4]: ./output_images/Warped_vs_unwarped_straight.png "Warp Example"
[image5]: ./output_images/Lane_lines_recogized_on_test6.png "Fit Visual"
[image6]: ./output_images/Projecting_back_with_text.png "Output"
[video1]: ./output_project_video.mp4 "Video"

### Reply to first review

Dear Reviewer, thank you very much for the helpful comments!

Note 1: Please find my new, corrected approach .ipynb in [P2_SDC_Ulrich_Voll_v2.ipynb](https://github.com/uv1000/P2/blob/master/P2_SDC_Ulrich_Voll_v2.ipynb).  

Note 2: Due to changes in the way the "Memory objects" are handled in my corrected approach the generation of the static images does no longer work the same as in the original .ipynb. Please refer to this .ipnb and the initial writeup (below) for everything concerning "single images".

Here is a [link to my video output_project_video_v2.mp4](./output_project_video_v2.mp4)

#### 1. Changes in Submission V2

- Fixed a bug in the way the last valid lane-parameters were visulized. I now plot the last valid values stored in the "Memory Objects" lLane and rLane. 

- Adapted the tresholding according to reviewers suggestions. 

- slightly reworked the "plausibility-check" concerning distance (mean instead of min/max)

- Turned off (sic!) the "compute from prior" variant, as it seems to produce a better video. 

- properly initialized the "Memory Objects" lLane and rLane

- adapted the hyperparameter alpha to 0.5 in the exponential moving average, in  function  plausibility_check().



#### 2. Discussion of Submission V2

- Having fixed the bug makes the image far more stable, in fact due to the bug I was alway plotting the "fresh" candidate lane-lines however implausible and wildly varying they were.

- The colour-thresholding has improved considerably, particurlarly on the yellow line. Not so much on the white (striped) lines. 

- There remains some (considerably reduced) amount of flickering when the binary-thresholding produces "patchy" patterns (in the shade). However some flickering might be acceptable to show that the image is "alive". From a safety perspective, exprapolating lines for too long in a situation where there (possibly) are no longer any lines may be dangerous.

- Some of the inaccuracies found by the first reviewer were due to the "hold on to last valid values" mechanism. I discovered it is much better to turn off the "compute from prior" section of the code. The code was working in the quizz, possibly a bug came in when integrating it in the project-.ipynb (if I could spot such a bug I would fix it, obviously). I am surprised that "compute from prior" should not do better than performing a histogram search  from scratch each time, but this is apparently the case.  

- I chose the smoothing hyperparameter alpha at 0.5 in order to compromise between lagging behind too much and being too sensitive in reacting to new flickering candidate-estimates. 

- The .ipynb still fails on the challenge videos. Not so badly as the first one, in that the estimates are no longer wildly flickering, but it still fails badly on the challenge videos. Only rarely new valid estimates are made, the projected lane-lines appear frozen inbetween. Moreover those rare new estimates classified as "valid" are wrong on many occations in the challenge videos.

- possible Improvement 1: Further improving on the thresholding. 

- possible Improvement 2: Try different averaging methods. 

- possible Improvement 3: Further elaborate on the plausibility check for new estimates. (More valid estimates that are truely valid, less "false positives"), possibly including a check with cv2.matchShapes

- possible Improvement 4: Search for and fix a bug in the "from prior" code (if there is any) (hard to believe using "from prior" should not help...)

- I am sceptical if the harder_challenge_video can be solved at all by this approach considering the narrow curves with lines disappearing behind the bonnet, also the light effects from looking into the sun ... 


#### 3. Reviewers Comments

1 Applying color threshold to the B(range:145-200 in LAB for shading & brightness changes and R in RGB in final pipeline can also help in detecting the yellow lanes.

2 And thresholding L (range: 215-255) of Luv for whites.

3 Check both the polynomials are correctly distanced with respect to the width of a lane.

4 Check the curvature of both polynomials whether it is similiar or not.

5 Also check the binary thresholding for improvement because mostly the detection is failing in shadows and brighter lane.

6 Also if there is an issue in a single frame then you can reject that wrong detection and reuse the confident detection from the previous detection. 

7 For smoothening the lane lines and for reducing wrong detection you can try averaging lane detection using a series of multiple frames.

8 In addition to other filtering mechanisms. You can also use cv2.matchShapes as a means to make sure the final warp polygon is of good quality. This can be done by comparing two shapes returning 0 index for identical shapes. You can use this to make sure that the polygon of your next frame is closer to what is expected and if not then can use the old polygon instead. This way you are faking it until a new frames appear and hence will get good results.

9 Annotated lines and patch on the rectified image maybe are not properly “unwarped” and annotated on the original images. This can be due to the lane lines not being clearly visible in the binary images.

#### 4. Replies to reviewers Comments

Comments/Points 1,2,5 above really helped, mainly for the yellow lines. 

Concerning Points 3,4,6 above: Please note that such mechanisms had been in place in V1, although there may have been bugs in the implementation. 

Concerning Point 8, I am not entirely sure what you mean by the "warp polygon". Are you suggesting I should perform an "adaptive clipping"? Ah, by comparing the shape (the green area) defined by the new values and the one defined by the old values using "match shape" when deciding if I should use the new values or the old ones, i.e. in the plausibility-check. Cf. my remark on "faking" things for too long from a safety perspective ... 

Concerning Point 9. I think the mapping for "unwarping" (back to the road) itself is ok and it works perfectly for static images. I believe the artefacts observed are due to "holding on too long" and, as you say, to the lane lines being incorrectly detected as the are not clearly visible (at least to the algorithm).  



## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I consider the rubric points individually and describe how I addressed each point in my implementation in [this .jpynb](https://github.com/uv1000/P2/blob/master/P2_SDC_Ulrich_Voll.ipynb).  

---

### 1 Camera Calibration

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

### 2 Pipeline (single images)

#### 2.1 Provide an example of a distortion-corrected image.

I implemented this in section 2.1 of my .jpynb.

Applying `cv2.undistort()` using the parameters/coefficients from above to test_images/test1.jpg yields:
![alt text][image2]

I chose not to write a separate wrapper-function around cv2.undistort() just for the sake of its own, in order avoid bloating the code.

#### 2.2 Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I implemented this in section 2.2 of my .jpynb.

In my function thresholding, I adapted the quiz .py file developed in the classroom. 

Thresholded Image 

= Binary picture 

= Logical OR of thresholding S value of HLS transform and Sobelx gradient. 

Details including threshold values see .jpynb. 

This yields:
![alt text][image3]

#### 2.3 Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

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

#### 2.4 Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I implemented this in section 2.4 of my .jpynb.

I adapted the sliding window algorithm from quizz/classrom for generating two 2nd order polynomial.  

I slightly adapted the functions find_lane_pixels() and fit_polynomial() from the quizz.

Mainly I had to adapt the height of y-values summed for the initializing histogram (lower 2/3 instad of lower 1/2, as in the classroom quiz/example). I payed to increase the parameter "margin" from 100 to 125 at some stage, but it is back to 100 in the present implementation.   

I also added some visualisation (paint lane found green), in fit_polynomial.

The resulting algorithm in Section 2.4 of the .jpynb now yields satisfactory results for all test images provided.

E.g. for image test6 I got:

![alt text][image5]

#### 2.5 Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

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
    
    

#### 2.6 Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this in section 2.6 of the .jpynb.

My function project_back(img,left_curverad, right_curverad,offset_x_m) projects the output image of pipeline step 2.4 (Fit lines with polynomial.) back onto the road, using a function warp_again() which is the inverse of the above warp(). 

My new helper-function warp_again()  is essentially interchanging src-points and dst-points when calling cv2.getPerspectiveTransform()) relative to warp() from pipline step 2.3 (Apply perspective transform.) 

Here is an example of my result on a test image:

![alt text][image6]

---

### 3 Pipeline (video)

#### Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).



Merely daisy-chaining the above steps yields a just about acceptable result on the simplest video (project_video), and totally fails on the challenge videos. 

To overcome these shortcomings, I introduced a "memory"-class as suggested. I made the following adapations. The overall structure of the code is not satisfactory in my eyes, but it somewhat does the trick. See discussion of shortcomings below. 

- I instantiated two (global?!) Objects of self-defined class Line().  ( Class Line() is more a struct than a class really, as there are not methods, presently). I am using far less properties as in the example suggested.

-  I provided a second function for computing lane-line coefficients from prior, along the lines of the respective quizz in the classroom. There are now two functions  find_lane_pixels_from_scratch(binary_warped) and  find_lane_pixels_using_prev(binary_warped). I decide upon which one to use upon a confidence value of the newly computed lane-line parameters. 

- The latter function reads out and uses the (valid! whenever this function ist called) values of the polynomial coefficients stored in the property .best_fit of the respective "memory"-class. 
    
    left_fit = lLine.best_fit
    right_fit = rLine.best_fit
    

- According to validity (using the class-property Line.valid, it has become a score now, really) I compute coefficients using the sooner or the latter of those two functions. I.e. if the last estimate of *both* lines was plausible, then the method "compute from prior" is applied.

- Later in the code (in a rather hidden place, admittedly) I check for plausibility/validity of the new coefficiens and adapt the class-property .valid and the memorized coefficients  in the property .best_fit accordingly.  If I get new plausible (and therefore considered valid) coefficients, the properties .best_fit and .valid of both lines are updated accordingly, this happens in my function plausibility_check(). 

-I call this function plausibility_check() in measure_curvature_and_offset_real(). Admittedly a spurious place but I would have had to massively redesign the existing codebase. At least at this point in the code all required information (distances in metres and the like) was available, and thats why I put it there. 

- Here is the code of the plausibility check, at an early stage, the conditions may get more refined/extended as I carry on. (I did indeed, please refer to the code for details.)

'def plausibility_check(ploty_cr, left_fitx_cr, right_fitx_cr,left_curverad, right_curverad, offset_x_m):
    
    mindistance= np.min(right_fitx_cr- left_fitx_cr) # at least 3m typ. value: 3.35317400005
    maxdistance=np.max((right_fitx_cr- left_fitx_cr)) # no more than 4m typ. value: 3.61941697379
    distance_ok= (mindistance > 2.9) and (maxdistance <3.8)
    isplausible_left=distance_ok   # to be implented. Are the new values any good?
    if isplausible_left:  # if yes update lLine / rLine 
        lLine.valid=True
        lLine.best_fit=left_fit
        lLine.allx = left_fitx
        lLine.ally = ploty
    else: # otherwise keep old values and mark invalid
        lLine.valid=False
        
    isplausible_right=distance_ok # to be implented. Are the new values any good?
    if isplausible_right:  # if yes update lLine / rLine 
        rLine.valid=True
        rLine.best_fit=right_fit
        rLine.allx = right_fitx
        rLine.ally = ploty
    else: # otherwise keep old values and mark invalid
        rLine.valid=False
    return isplausible_left, isplausible_right '

- Note that the actual update of the "memory"-class Line() is performed implicitely as as side-effect on a global Variable. I know this is not perfect, ok it is messy ... 

- In the memory structure there should be always a valid encoding of lane lines. The visulisation always plots this last valid encoding. This somehow does not always work. 

- I performed smoothing, using an exponential moving average 
new value = (1-alpha) * old_value + alpha * new_value
This performs smoothing like in a PT1-Filter. Only plausible new parameters are allowed to contribute as "new_value"s.
I suspect that this is not working as expected, see below. 


Here's a [link to my video result](./output_project_video.mp4)


### 4 Discussion

####  Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The algorithm performs just about acceptable on the simplest video (project_video.mp4).

It badly fails on the two challenge videos, and I see no obvious way to improve on them, presently.

Possibly this is related to the aforementioned problem that there should not be any non-plausible line-encodings stored in the "Memory-Objects" lLine/rLine. Only plausible new line-encodings are allowed to go into the smoothing. And only the smoothed values in the "Memory-Objects" are being plotted (at least this is what I indended). 

However, in the challenge videos the plotted lanes jump and whiggle around. This is an indication, that this "select plausible and smoothe" approach does not work as intended.  

The code should be massively redesigned, more OO design. The classes should have methods for updating them and the like.





