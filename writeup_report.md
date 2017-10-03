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
[image8]: ./writeup_files/hsl.png "HSL Warp Image"
[image9]: ./writeup_files/s_channel.png "Sobel and S-Channel Image"
[image10]: ./writeup_files/warp_roi.png "Sobel ROI"
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

The pipeline code is available in `pipeline()` function, in the *Pipeline* subsection.

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one *(Pipeline line 11)*:
![Undistort and original image][image5]

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

I warped the original image, following the source and destination points *(Pipeline line 14)*:

![Warp Image][image7]

#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I converted the image from RGB color space to HSL (hue, saturation, lightness) color space *(Pipeline line 17)*.

![HSL Warp Image][image8]

To view the best result after apply a image filter, I tested two algorithms: Sobel and Binary S-Channel. In my first approach, I prefered to use Binary S-Channel, but the algorithm didn't find the line lanes correctly. Then I combined S-Channel with Sobel and the results were better, but not good enough. Then I tested Sobel algorihtm alone - this procedure had good results for different situations. The following image contain the comparison of S-Channel and Sobel algorithms.

![S-Channel and Sobel Image][image9]

As Sobel algorithm returned better results, I kept it only *(Pipeline line 24)*. Sobel algorithm uses the following approachs:
* Directional Thresh
* Magnitude Thresh
* ABS Thresh (X direction only)

I tuned the parameters of Sobel algorithm to improve the results. The directional thresh has none or little important in the final result. The majors thresh contributions were ABS and Mag Thresh. Using ABS Thresh in Y direction, the result was worse then Binary S Channel approach.

To improve the results above, I cutted of some parts of the image after apply Sobel algorithm to help the 2nd order polynomial function discovers the lane line. The image was cutted near the lane lines expected positions to increase the performance and discard some other information that can mislead the algorithm *(Pipeline line 26-28)*. The results is shown below.

![Sobel ROI][image10]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I did a polynomial function, the same as Udacity tutorial, and fit my lane lines with a 2nd order polynomial kinda like this:

![Plot 2nd order polynomial][image11]

Then I ploted the polynomial over the warped image *(Pipeline line 32)*.

![Polynomial plot over warped image][image12]

Next, I draw a green area inside the combination of my polynomial plot and my ROI *(Pipeline line 38-43)*.

![Green area over warped image][image13]

Then, I unwarped the green area image and returned to the original image *(Pipeline line 46)*.

![Green area over unwarped image][image14]

As it sees, in the last image, the unwarped procedure lost some data information. Then, I combine the unwarped green image with the original image and recovery some data out of my region of interest (top of the image), making final output transform *(Pipeline line 49)*.

![Final output transform][image15]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To calculate the curvature of the lane, I used the same model that was showed in the Udacity tutorial, with the conversion pixels to meters. The code was available in the `curvature()` function, in the *Calculate Curvature and Relative Positions Functions* subsection.

The curvature was calculated using an intermediary image with the green area marked over. Then I used Sobel transformation and did the 2nd order polynomial. To improve the result I chosed the left lane (constant) as the best to approach the result *(Curvature function lines 5-15 / Pipeline lines 51-60)*.

To calculate the position of vehicle with respect to center, I pick I pixel inside the green area (over **y axis**) and the returned the value for **x axis** in the left and right curve. Then, I multiplied the value of the results by the constant of conversion pixels-meters, as shown in formula below: 

`
((left_pos) - (1280-right_pos)) * xm_per_pix
`

The code is available in the function `relative_position`, in the *Calculate Curvature and Relative Positions Functions* subsection *(Relative_position function line 18 / Pipeline lines 51-60)*.

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

Then, I made the images transformations. The most difficult was to developed the right procedure was to warp the image, and get the correct final result for all images. I used many approachs to achieve my goal.

##### Previous Development procedures

This section contain different approachs that I used but didn't result in correctness.

* First solution was transform the image using the Region of Interest (ROI - part of the original image without warp) in a image of 720x1280. Then I warped the image and marked my green area. When I unwarped the image back, I lost some information of the picture, resulting in black areas over the original image, not only the green space in my ROI. To recover those information, I tried to fill each pixel outside my ROI (a polygon) with the original image. The result worked, but was too slow.
* Thinking in other solution, I decided to shrink my destination area in the warp transformation, instead of create a image of 720x1280. Then, I cropped my region of interest and applied some transformation just in this area. Next, I combined my marked green area to the warped image. Finally, I unwarped the combined image. After this procedure, I lost some data just on top of the image, easier to recover then my first approach.
* Other approach that I tried was warp the image before define my Region of Interest. This approach was used in the final output. But, processing the image, I tried many different things, that didn't result well, as:
	* Use Binary S-Channel;
	* Use combination of Binary S-Channel and Sobel;
	* Use Sobel with Y direction ABS transformation;
	* Crop ROI before applied Sobel;
	* Do not convert to HSL (combined with all above);
	* Other approachs mixing all above.

##### Development procedures and problems in development
The major problem was to process image and remove the noise. I used many approachs as explain in the last section. Some approachs results in closer results and tunning operations cause lot of time spent. I tunned many of aproachs described until I realized that I can't improve the results anymore. These wrong approachs consume most of the development time (many, many, many time).

##### Increase performance possibilities
To improve my model, I think I can use the Sobel combined with some other image transformations like Canny or Hough. I can try to tune Sobel parameters with different threshs.

One possible improvement is use the continuous lane line to predict the dashed line, based in some dashs, improve the position of the vehicle on track and the 2nd order polynomial.

Other possible approach is try to detect colors in the image, like yellow and white colors. Then, apply Sobel. This approach helps to find something that I already expect and improve the results of Sobel operation.
