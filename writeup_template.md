## Project 4
### Advanced Lane Finding

---

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

[preCalibration]: ./output_images/pre-calibration.png "preCalibration"
[postCalibration]: ./output_images/post-calibration.png "postCalibration"
[sobelX]: ./output_images/sobelX.png "sobelX"
[sChannel]: ./output_images/sChannel.png "sChannel"
[combinedBinary]: ./output_images/combinedBinary.png "combinedBinary"
[linesNormalPerspective]: ./output_images/linesNormalPerspective.png "linesNormalPerspective"
[linesTransformPerspective]: ./output_images/linesTransformPerspective.png "linesTransformPerspective"
[windowFit]: ./output_images/windowFit.png "windowFit"
[previousLineFit]: ./output_images/previousLineFit.png "previousLineFit"
[highlitedLanes]: ./output_images/highlitedLanes.png "highlitedLanes"


[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image. 

To begin, we create the object points, which are the "real world" coordinates of the chessboard corners. Since we assume the chessboard is on a flat plane with z=0, these points are just (x, y) where x goes from 0 to 8 and y from 0 to 5.

We then use cv2's function to find the chessboard corners in the image. If successful, we save a copy of those coordinates along with a copy of the "real world" coordinates. 

Once we have all the coordinates from all the calibration images, we pass them along to cv2's calibrateCamera function to calculate the calibration matrix. Using this matrix we can undistort any image taken by the camera.

Here is one of the chessboard images before and after being undistorted:

![before calibration][preCalibration]
![after calibration][postCalibration]

### Pipeline (single images)

#### 1. Undistort the image

The first step is to undistort the image. This is done by using the calibration matrix obtained previously. See above.

#### 2. Threshold the image

To get a thresholded binary I used a combination of sobel and saturation channel.

For the sobel piece, I used the sobel in the x direction. This identified pixels where there is a large change in the pixel value in the x direction. This will be true along edges.

For the saturation channel, I first converted the image from RGB to HLS. Then I isolated the S channel. This does a good job of finding the lane lines even when they are in shadows.

You can see in the following images that sobel finds the edges of the lane lines while the S channel finds the middle. Adding the two together yields a good selection of the lines.

Sobel:

![sobel X][sobelX]

S channel:

![s channel][sChannel]

Combination:

![combined binary][combinedBinary]

#### 3. Perspective transform

To accomplish the perspective transform, we assume the road is flat and the camera stays stationary on the car throughout the video. I used one of the test frames and manually identified the pixel locations of the start and stop of the lane lines. Like this:

![Lines before perspective transform][linesNormalPerspective]

I then defined the destination points to be a box that goes from the top to bottom of the image. This produces an image where the lines appear parellel like this:

![Lines after perspective transform][linesTransformPerspective]


#### 4. Identify the lines

To identify the pixels that make up a lane line I first needed to know where to look for those pixels. For the first time finding a line, I do this by finding the point on the bottom of the image with the highest number of active pixels (in the binary threshold image). I start at that point and use a box 200 pixels wide and 80 pixels tall. I count all the active pixels in that box as part of the line. I then move to a box on top of the previous one, centering this box on the average position of the active pixels in the previous box. I do this until we reach the top of the image. Here is an image the illustrates this with the windows drawn in green, the lane pixels colored, and the corresponding fit drawn:

![lane being found with windows][windowFit]

If we are processing a video, then we know that the lane lines aren't going to move too much from frame to frame. That means instead of using the window approach, we can just use the position of the lane from the last time we found it. I count all the active pixels on within 100 pixels of the previous line as part of the current line. Here is an image showing the window from the previous fit we used to identify lines in this image:

![lane being found from previous fit][previousLineFit]

Once we have identified all the points that are part of the lane lines, we simply use numpy's polyfit function to fit a second degree polynomial to them.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Plotting the lanes on the original image

To draw the lanes on the original image, I start by drawing the fit on a blank image in the warped perspective. I use cv2's fillConvexPoly to draw the fit. Then I use the inverse perspective matrix to transform the fit to the original perspective instead of the warped one. Now I have an image that is just the lane, so I use cv2.addWeighted to add that image on top of the original one. The end result looks like this:

![lanes highlited on image][highlitedLanes]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
