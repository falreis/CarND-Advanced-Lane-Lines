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

[image1]: ./writeup_files/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./writeup_files/binary_combo_example.jpg "Binary Example"
[image4]: ./writeup_files/warped_straight_lines.jpg "Warp Example"
[image5]: ./writeup_files/color_fit_lines.jpg "Fit Visual"
[image6]: ./writeup_files/example_output.jpg "Output"
[video1]: ./output_project_video.mp4.mp4 "Video"

---

#### 1. Writeup / README

My project includes the following files/folders:
* [code.ipynb](./code.ipynb) containing jupyter notebook file, with the code of this project
* [/output_images](./output_images/) folde contain all the result image files (calibrations and test images)
* [output_project_video.mp4](./video.mp4) containing the output video file
* [writeup_report.md](./writeup_report.md) summarizing the results

#### 2. Functional code
To view the code and execute it, it's necessary Jupyter Notebook module with CarND-Term1 Anaconda environment. It's possible to see the code in Github too.

