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
[image13]: ./writeup_files/green_area_warp.png "Green area over warped image"
[image14]: ./writeup_files/green_area_unwarped.png "Green area over unwarped image"
[image15]: ./writeup_files/final_output_transform.png "Final output transform"
[image16]: ./writeup_files/.png " "
[image17]: ./writeup_files/.png " "
[image18]: ./writeup_files/.png " "
[image19]: ./writeup_files/.png " "
[image20]: ./writeup_files/.png " "
[video1]: ./project_video.mp4 "Video"

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cells of the subsection "Calibration Images" of the IPython notebook located in "./code.ipynb". The subsection uses some functions developed in the section "Camera Calibration Functions", located in the beginning of the file.

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

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
!["Undistort and original image"][image5]

#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Firstly, it was necessary to define a region of interests (ROI), cropping everything that I don't need in the image. 

![Region of Interest (ROI) of the image][image6]

Then, I proceded the code for my perspective transform includes a function called `warp_transform()`, which appears in the section *Warp Transform Functions*
(output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warp_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
vertices_src = np.float32(
    [[ 270, 700],
     [ 550, 450],
     [ 730, 450],
     [1100, 700]]
)

vertices_dst = np.float32(
    [[ 550, 700],
     [ 550, 450],
     [ 730, 450],
     [ 730, 700]]
)
```

This resulted in the following source and destination points. The results are based in ROI points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 270, 700      | 550, 700      | 
| 550, 450      | 550, 450      |
| 730, 450      | 730, 450      |
| 1100, 700     | 730, 700      |

![Warp Image][image7]

![Warp Image and crop of ROI][image8]


#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I croped my ROI in the image, then I convert the color to HSL (hue, saturation, lightness)

![HSL ROI Image][image9]

I ignored the Hue and Lightness of the output image and keep only the Saturation (S) channel. I used a combination of color and gradient thresholds to generate a binary image.

![S-Channel Image][image10]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![Plot 2nd order polynomial][image11]

Then I ploted the polynomial over the warped image.

![Polynomial plot over warped image][image12]

Next, I draw a green area over my polynomial plot and my ROI.

![Green area over warped image][image13]

Then, I unwarped the green area image and returned to the original model.

![Green area over unwarped image][image14]

As it sees, in the last image, the unwarped procedure lost some data information. Then, I combine the unwarped green image with the original image and recovery some data out of my region of interest, making final output transform.

![Final output transform][image15]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
