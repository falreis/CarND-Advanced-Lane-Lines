## Advanced Lane Finding Project

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
[image1]: ./writeup_files/calibration2.jpg "Original calibration image"
[image2]: ./writeup_files/chessboard.png "Find chessboard corners"
[image3]: ./writeup_files/calibrate_undistort.png "Calibrate and undistort image"
[image4]: ./writeup_files/warped_cal.png "Calibration image warp"
[image5]: ./writeup_files/undistort_image.png "Undistort and original image"
[image6]: ./writeup_files/roi.png "Region of Interest (ROI) of the image"
[image7]: ./writeup_files/warp_image.png "Warp Image"
[image8]: ./writeup_files/warp_roi.png "Warp Image and crop of ROI"
[image9]: ./writeup_files/hsl.png "HSL ROI Image"
[image10]: ./writeup_files/s_channel.png "S-Channel Image"
[image11]: ./writeup_files/plot_continuous.png "Plot 2nd order polynomial"
[image12]: ./writeup_files/polynomial_warp.png "Polynomial plot over warped image"
[image13]: ./writeup_files/green_area_warped.png "Green area over warped image"
[image14]: ./writeup_files/green_area_unwarped.png "Green area over unwarped image"
[image15]: ./writeup_files/final_output_transform.png "Final output transform"
[image16]: ./writeup_files/final_image.png "Final output image"
[video1]: ./project_video.mp4 "Video"

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cells of the subsection "Calibration Images" of the IPython notebook located in [./code.ipynb]. The subsection uses some functions developed in the section "Camera Calibration Functions", located in the beginning of the file.

I started by using a random calibration image from the calibration examples. For example, here, we have the image *calibration2.jpg*, from *camera_cal* folder.

![Original calibration image][image1]

Then I perform some steps, to undistort and warp the image, as it follows:

* Find some chessboard corners and draw it (***chessboard()*** custom function)

The cv2 functions used was `cv2.findChessboardCorners` and `cv2.drawChessboardCorners`.

![Find chessboard corners][image2]

* Calibrate the camera and undistort image (***calibrate_camera()*** and ***undistort()*** custom functions)

I used the output `objpoints` and `imgpoints` with the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied the distortion correction to the test image using the `cv2.undistort()`)* 

![Calibrate and undistort image][image3]

* To test and see the results of the camera calibration, I unwarped the output image.

I used the `cv2.getPerspectiveTransform` and `cv2.warpPerspective` to do that.

![Calibration image warp][image4]


### Pipeline (single images)

The pipeline code is available in function `pipeline()`, in the *Pipeline* subsection.

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
!["Undistort and original image"][image5]

#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I proceded the code for my perspective transform includes a function called `warp_transform()`, which appears in the section *Warp Transform Functions*
(output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warp_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner *(Warp transform subsection - line 1-13)*:

```python
vertices_src = np.float32(
    [[ 120, 720],
     [ 500, 500],
     [ 700, 500],
     [1200, 720]]
)

vertices_dst = np.float32(
    [[ 500, 720],
     [ 500, 0],
     [ 620, 0],
     [ 660, 720]]
)
```

This resulted in the following source and destination points. 

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 120, 720      | 500, 720      | 
| 500, 500      | 500, 0        |
| 700, 500      | 620, 0        |
| 1200, 720     | 660, 720      |

I warped the original image, following the source and destination points *(Pipeline line 12)*:

![Warp Image][image7]

Then I cropped the image in the Region of Interests (ROI) points *(Pipeline line 14-16)*. This region contains only the track, and put all information around in black pixel.

![Warp Image and crop of ROI][image8]


#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I changed my ROI image from RGB color space to HSL (hue, saturation, lightness) color space. *(Pipeline line 5)*

![HSL ROI Image][image9]

I ignored the Hue and Lightness of the output image and keep only the Saturation (S) channel. I used a combination of color and gradient thresholds to generate a binary image. *(Pipeline line 9)*

![S-Channel Image][image10]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I did a polynomial function, the same as Udacity tutorial, and fit my lane lines with a 2nd order polynomial kinda like this:

![Plot 2nd order polynomial][image11]

Then I ploted the polynomial over the warped image *(Pipeline line 17)*.

![Polynomial plot over warped image][image12]

Next, I draw a green area inside the combination of my polynomial plot and my ROI *(Pipeline line 20-30)*.

![Green area over warped image][image13]

Then, I unwarped the green area image and returned to the original image *(Pipeline line 32)*.

![Green area over unwarped image][image14]

As it sees, in the last image, the unwarped procedure lost some data information. Then, I combine the unwarped green image with the original image and recovery some data out of my region of interest, making final output transform *(Pipeline line 35)*.

![Final output transform][image15]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To calculate the curvature of the lane, I used the same model that was showed in the Udacity tutorial, with the conversion pixels to meters. The code was available in the `curvature()` function, in the *Calculate Curvature and Relative Positions Functions* subsection.

To calculate the position of vehicle with respect to center, I calculate the mean of `left_fitx` value, mesure the distance to the center of the image and multiply to the constant of conversion pixels-meters. The code is available in the function `relative_position`, in the *Calculate Curvature and Relative Positions Functions* subsection.

The code calling for the both custom functions are available in the *Pipeline* function, lines 37-42.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The final image of *Pipeline* function can be seen below.

![Final output image][image16]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

##### Development procedures and issues that I faced in my implementation

To make the project, I started trying to calibrate the camera and writing some functions, that can be usefull in other situations.

Then, I made the images transformations. The most difficult procedure was to warp the image, get the S channel and the mark a green area in my region of interest. 

* The problem was that I transformed the image using the Region of Interest in a image of 720x1280. Then I warped the image and marked my green area. When I unwarped the image back, I lost some information of the picture, resulting in black areas over the original image, not only the green space in my ROI. To recover those information, I tried to fill each pixel outside my ROI (a polygon) with the original image. The result worked, but was too slow.
* Thinking in other solution, I decided to shrink my destination area in the warp transformation, instead of create a image of 720x1280. Then, I cropped my region of interest and applied some transformation just in this area. Next, I combined my marked green area to the warped image. Finally, I unwarped the combined image. After this procedure, I lost some data just on top of the image, easier to recover then my first approach.

##### Development procedures and problems in development
The pipeline built has some problems when exist different colors of the road (asphalt), specially when the aspalth is dark and become clear. For example, in the 25th second of the video, the pipeline loose the track for a moment then get back, finding the correct track.

##### Development procedures and problems in development
One procedure that not worked as I expected was the use of Sobel transform. The sobel transform results mostly in worst results that I could do using only the S channel.

To improve my model, I can use the S channel combined with some other image transformations like Canny or Hough. I can try one more time use Sobel with different thresh.

Other thing that is possible to do is use the continuous lane line to predict the dashed line, based in some dashs, improve the position of the vehicle on track and the 2nd order polynomial.
