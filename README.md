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

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it! (nothing better than this from the template :-))

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in IPython notebook "Project 2.ipynb". (Cells In[17])

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![original image][./output_images/original_chessboard.png]
![undistorted image][./output_images/undist_chessboard.png]

### Pipeline (single images)

All the code is available in the Project 2 IPython notebook. (It also has all the images from the steps). The process_image function is the high level function implementing the pipeline for images. It takes help from several helper functions (implementations of the various steps and support functions). 

All the output images are also available in the output_images folder. 

The project video output is project_video_output.mp4.

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
I used the camera calibration and distortion coefficents from the calibration steps earlier and using opencv method cv2.undistort() function undistorted the input image

Original and Undistorted image below

![Original Image][./output_images/original_test_image.png]
![Undistorted][./output_images/undist_test_image.png]

The code is available in cells In[18]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In addition to implementing and trying out the Sobel gradients and S channel (from HLS transformed copy of the image) color thresholds I also attempted to see the results of applying RGB and L channel thresholds. 
Sobelx gradient (as expected) was the most promising of the Sobel gradients. SobelY, directional gradients and a combination of these did not yield anything significantly more promising. 

Of all the color transformation and thresholding and a combination of all I decided finally to go with only s-channel thresholds based on visual inspection of the output images. (In hindslight need to come back to this step to make the challenge video work!!) 

![Sobel gradient X][./output_images/SobelX.png]
![S Channel ][./output_images/s_channel.png]
![Combined][./output_images/combined_image.png]

The code is available in cells In[34], In[19]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I chose to hardcode the source and destination points in the following manner. I applied the undistort and gradient/color thresholds to get the binary image with lanes identified for the test image - straight_lines1.jpg. 

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

The perspective tranform matrices M and Minv obtained were used in the process_image pipeline function. 

![Source and destination points][./output_images/src_dst_points.png]
![Binary Image][./output_images/combined_image.png]
![Perspective Transform][./output_images/warped.png]

The code is available in cells In[36]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The high level flow is as follows
 1. Use sliding window method for the first time
 2. Use search around line method from the next time onwards
     The function find_lanes helps orchestrates this based on the switch fresh_search
 3. Do a simple sanity of the lines detected (check_lines function)
     a)Check if anyline came back
     b)If so check if the difference between left and right (horzontal distace of the lane) is comparable to the average      difference so far and discard if there is a huge variation. 
 4. If the line is not good try sliding window method if not done already
 5. If the line is good then average it out over the last 10 detected lines and use it. (Need to check how this behave if there are sharp curves). The average_line function helps in this purpose. The rubrics point sugestion to use a weighted average helped in good "smoothing" effect.
 

![sliding window result][./output_images/sliding_window.png]
![search around line result][./output_images/search_around_line.png]

The code is available in cells In [41] and In[42]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The left and right lane radius of curvature was determined using the helper function measure_radius_of_curvature.
The basic assumptions was to consider the lane is about 30 meters long and 3.7 meters wide. Using the second order polynomials detected for left and right lanes calculated the real radius of curvature of the detected left and right lanes. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The result of running the process_image pipeline function is given below.

![original image][./output_images/orig_image.png]
![processed image][./output_images/processed_image.png]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. 

I spent a lot of time experimenting with the gradients and various color thresholds and checking the visual output to see if it makes sense. I spent considerable time seeing if I can reduce the noise levels around the roads (trying to isolate only the lane lines). One late realization was that when I did the perspective transform it automatically removed the surroundings and highlighted only the lanes. I plan to do some more work on the color thresholds to get a better output based on this. (In hindsight I think I spent more time here than completing the pipeline and trying it out on various images/videos). 
I am sure that I need to enhance the color thresholds to get the challenge video working. Right now the shadow lines are being detected as actual lanes, the yellow lines under bright light is not getting detected accurately always. 
I also need to check what happens if there are sharp curves. The averaging part may have to became more sophisticated to handle this. 


