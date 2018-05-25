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

To identify the pixels that make up a lane line I first needed to know where to look for those pixels. For the first time finding a line, I do this by finding the point on the bottom of the image with the highest number of active pixels (in the binary threshold image). I start at that point and use a box 200 pixels wide and 80 pixels tall. I count all the active pixels in that box as part of the line. I then move to a box on top of the previous one, centering this box on the average position of the active pixels in the previous box. I do this until we reach the top of the image. Here is an image that illustrates this with the windows drawn in green, the lane pixels colored, and the corresponding fit drawn:

![lane being found with windows][windowFit]

If we are processing a video, then we know that the lane lines aren't going to move too much from frame to frame. That means instead of using the window approach, we can just use the position of the lane from the last time we found it. I count all the active pixels within 100 pixels of the previous line as part of the current line. Here is an image showing the window from the previous fit we used to identify lines in this image:

![lane being found from previous fit][previousLineFit]

Once we have identified all the points that are part of the lane lines, we simply use numpy's polyfit function to fit a second degree polynomial to them.

#### 5. Radius of curvature and distance from center

To calculate lengths from the images we have to be able to convert from pixels to meters. We know that a typical lane in the US is 3.7 meters wide and the camera captures about 30 meters of it. In the tranformed perspective, the lane takes up the whole height of the image (720 pixels) and we set up the transform so it is 680 pixels wide. Using these numbers we get the pixel to meter ratio.

From there it is a simple matter of transforming the pixel values to meter values and re-calculating the polynomial fit in meter dimensions. Then we can use algreba to find the radius of the curve in meters. I found that this value is a lot more variable when the lane is straight. This makes sense since a straight line should have an infinite radius, so slight changes in the line will cause large swings in the calculated radius.

For the distance from center, we find what the x values are for the right and left lanes at the bottom of the image. Then we find the value between the two of them and compare that to the center of the image.

#### 6. Plotting the lanes on the original image

To draw the lanes on the original image, I start by drawing the fit on a blank image in the warped perspective. I use cv2's fillConvexPoly to draw the fit. Then I use the inverse perspective matrix to transform the fit to the original perspective instead of the warped one. Now I have an image that is just the lane, so I use cv2.addWeighted to add that image on top of the original one. The end result looks like this:

![lanes highlited on image][highlitedLanes]

---

### Video Results


Here's a [link to my video result](./output_images/video.mp4)

---

### Potential improvements

My lane finding pipeline could definitely use some improvements.

Speed is definitely a concern. It takes almost a minute to process the video. If this pipeline is going to be used in a car for real time decision making it needs to perform as fast as possible.

I am not smoothing out the calculated lane across frames. I could use the average values calculated across the last several frames to come up with the actual lane lines for each frame. That would help reduce some of the noise and jitters.

I found that the s channel thresholding was usually effective. But there were times when it was noisy. Specifically near the end of the video where there is a shadow across two the lane where the lane changes color. I tried to reduce the noise by ignoring the s channel when the middle of the image was too bright, but a more robust solution would be better.

Looking at one of the challenge vidoes, my pipeline has problems when there is a curb or barrier next to the lane. The video thinks the line at the bottom of the barrier is a lane line. The original video didn't have this problem becuase the barrier is farther away and the perspective transform doesn't include it. I need a more intelligent way of doing the transform to pick out the lines without a human having to manually specify pixel values. I also could maybe identify every line in the image and then intelligently decide which ones are likely to be lane lines.
