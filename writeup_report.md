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
[image6]: ./writeup_files/warp_concatenate.png "S-Channel + Sobel (Left) and Sobel (Right)"
[image7]: ./writeup_files/warp_image.png "Warp Image"
[image8]: ./writeup_files/hsl.png "HSL Warp Image"
[image9]: ./writeup_files/s_channel.png "Sobel and S-Channel Image"
[image10]: ./writeup_files/warp_roi.png "Warp ROI"
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

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one *(Pipeline line 19)*:

![Undistort and original image][image5]

#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I proceded the code for my perspective transform includes a function called `warp_transform()`, which appears in the section *Warp Transform Functions*
(output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warp_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner *(Warp transform subsection - line 4-16)*:

```python
vertices_src = np.float32(
    [[ 200, 720],
     [ 350, 460],
     [ 850, 460],
     [1200, 720]]
)

vertices_dst = np.float32(
    [[ 450, 720],
     [ 0, 0],
     [ 1100, 0],
     [ 800, 720]]
)
```

This resulted in the following source and destination points. 

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 450, 720      | 
| 350, 460      | 0, 0          |
| 850, 460      | 1100, 0       |
| 1200, 680     | 800, 720      |

I warped the original image, following the source and destination points *(Pipeline line 22)*:

![Warp Image][image7]

#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I converted the image from RGB color space to LUV and LAB color spaces *(Pipeline line 26-27)*.

![LUV and LAB Warp Images][image8]

The LAB color space is good for discover yellow lines while LUV color space is good to find white lines. Combining both color spaces (for situations with 2 white or yellow lines), it's possible to find the best algorithm to find the lane lines.

To improve the results of the color spaces, I decided to use Binary S-Channel algorithm, with some tuning. The results was as good as Sobel algorithm applied in these color spaces *(Pipeline line 30-31)*.

![Binary S-Channel for LUV and LAB color spaces][image9]

I merge both images and generated combined image, as shown below *(Pipeline line 32)*.

![Combined Image][image6]

To avoid processing and mistakes, I cutted some parts of the image after apply Binary S-Channel algorithm. The image was cutted near the lane lines expected positions to increase the processing performance *(Pipeline line 35-36)*. 

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I did a polynomial function, the same as Udacity tutorial, and fit my lane lines with a 2nd order polynomial kinda like this:

![Plot 2nd order polynomial][image11]

Then I ploted the polynomial over the warped image *(Pipeline lines 39-42)*.

![Polynomial plot over warped image][image12]

Next, I draw a green area inside the combination of my polynomial plot and my ROI *(Pipeline line 66-79)*.

![Green area over warped image][image13]

Then, I unwarped the green area image and returned to the original image *(Pipeline line 82)*.

![Green area over unwarped image][image14]

As it sees, in the last image, the unwarped procedure lost some data information. Then, I combine the unwarped green image with the original image and recovery some data out of my region of interest (top of the image), making final output transform *(Pipeline lines 85-87)*.

![Final output transform][image15]

As an aditional feature to improve performance, I calculate the 2nd order polynomial with sliding widows only to some frames. I calculate the position and keep it in memory. To the next frames, I search near the position of the frame saved. With this, the algorithm increase performance. After an amount of frames, 10 frames for example, I run sliding window procedure again to keep on track and renew the save position *(Pipeline lines 39-42)*.

Other improvement is to store the last found lane inside a variable. If the algorithm was unable to find the lane in the current frame or the correct mesurement curve radius, it uses the last one to keep in track. This solution keeps the stability of the algorithm *(Pipeline lines 45-60, 90-96)*.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To calculate the curvature of the lane, I used the same model that was showed in the Udacity tutorial, with the conversion pixels to meters. The code was available in the `curvature()` function, in the *Calculate Curvature and Relative Positions Functions* subsection.

The curvature was calculated using the curvature for the warped image. The constants needed to be updated, because of the warp transformation. The constant values was converted to the area of road after the warped transformations. To improve the results I calculated the mean value between the left and the right lane curvature, to find the correct lane curvature *(Curvature function lines 8-18 / Pipeline lines 90-91)*.

To calculate the position of vehicle with respect to center, I pick I pixel in the side of the green area, near the car, (over **y axis**) and the returned the value for **x axis** in the left and right curve. I also divided the image in 2 parts, the left and the right. I also multiplied the value of the results by the constant of conversion pixels-meters, as shown in formula below: 

`
  (IMAGE_WIDTH - (left_pos + right_pos)) * xm_per_pix
`

The code is available in the function `relative_position`, in the *Calculate Curvature and Relative Positions Functions* subsection *(Relative_position function lines 21-24 / Pipeline line 99)*.

##### Note
As the image used in the pipeline has size of 1280x720, the values returned from the 2nd order polynomial for x axis are inside the range 0-1280. If the algorithm returns a point in the left lane, it's X axis will be 600 (for example) not 0. The right lane will return 850 (for example) not 250.

If the algorihtm doesn't use range of 0-1280, it will be necessary to normalize the values to calculare the curvature and the relative position.

In previous development versions, I used an unwarped image to find and mesure the curvatures. The size in this previous version was relative to the size of the lane at the position of the axis y=650 (about 700 pixels).

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
	* Clahe algorithm
* Other approach that worked well but had some limitations was convert the algorithm to HSL channel and apply 2 algorithms: Sobel and Binary S-Channel. Combining S-Channel with Sobel and the results were better, mostly for the left lane. Using Sobel algorihtm alone, the results were good for different situations mostly to identify right lines. Combining left and right lanes together, the result were good, but not good enought for all situations.

##### Development procedures and problems in development
The major problem was to decide the best approach to solve the problem. I had used many approachs as I explained in the last section. Some approachs results in closer results and tunning operations cause lot of time spent. I tunned many of aproachs described until I realized that I can't improve the results anymore. These wrong approachs consume most of the development time (many, many, many time).

##### Increase performance possibilities
To improve my model, I think I can use the current algorithm combined with some other image transformations like Canny or Hough.

One possible improvement is use the continuous lane line to predict the dashed line, based in some dashs, improve the position of the vehicle on track and the 2nd order polynomial.

