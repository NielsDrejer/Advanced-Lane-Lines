## Term 1 Project 4 Advanced Lane Finding

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

The results of the project is a video called project_output_video.mp4, the python code in the Jupyter Notebook file 'Project 4 Advanced Lane Finding.ipynp' and the project writeup in this file.

[//]: # (Image References)

[image1]: ./output_images/calibration2_corners.jpg "Undistortion"
[image2]: ./output_images/calibration2_undistorted.jpg "Undistotion"
[image3]: ./output_images/Undistortion.png "Undistortion"
[image4]: ./output_images/Thresholding.png "Thresholding"
[image5]: ./output_images/Cropping.png "Cropping"
[image6]: ./output_images/Warping.png "Warping"
[image7]: ./output_images/SlidingWindows.png "Histogram"
[image8]: ./output_images/SlidingWindows2.png "Sliding Windows"
[image9]: ./output_images/KnownLines.png "Known Lines search"
[image10]: ./output_images/FinalResult.png "Final Result"

[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

Camera calibration is done in Part 1.1 of the Jupyter Notebook file 'Project 4 Advanced Lane Finding.ipynp'. I use the chessboard images provided in the project material. First I identify the corners using the cv2.findChessboardCorner function. This results in images like this:

![alt text][image1]

Of the 20 chessboard images my code found corners on 17. This is enough to collect object and image points for calculation of the camera matrix and the distortion coefficients. This is done with the function cv2.calibrateCamera. Using these parameters and the function cv2.undistort images can now be undistorted. The undistorted version of the above chessboard image looks like this:

![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The test images supplied in the project material are undistorted in Part 1.2 of the Jupyter Notebook file 'Project 4 Advanced Lane Finding.ipynp'.

An example of the result looks like this:

![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In Part 1.3 of the the Jupyter Notebook file 'Project 4 Advanced Lane Finding.ipynp' I write code to create binary thresholded images. This area is one where a lot of experimentation is required. I decided to reuse the knowledge I obtained in the first project of this term, and apply a color transformation to HLS color space using cv2.cvtColor. A combination of H and S thresholds are used to identify the yellow colors in the image, and an L threshold is used for the white colors. This worked well in project number 1.

Furthermore I apply an absolute Sobel threshold and a direction of Sobel gradient threshold, using the cv2.Sobel library function. In both cases I use a kernel size of 15. Also in both cases I had to experiment with the threshold values to avoid getting too much white.

My final implementation is found in the function get_thresholded_image. An example of how this function works can be seen here:

![alt text][image4]

The second part of processing of the thresholded image is to crop drop it to get only the parts in which lanes are visible. Also here I reused what I did in project 1 for the function crop_to_area_of_interest. This function removes everything outside of a polygon using cv2.fillPoly and produces results like this:

![alt text][image5]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Next step is to perform a perspective transformation to get a birds eye view of the images. This is done in In Part 1.4 of the the Jupyter Notebook file 'Project 4 Advanced Lane Finding.ipynp'.

I chose the source and destination points for the transformation by manual inspection of the test images with straight lines to be:

| Source        | Destination   |
|:-------------:|:-------------:|
| 200, 720      | 320, 720        |
| 1110, 720      | 920, 720      |
| 565, 470     | 320, 1      |
| 720, 470      | 920, 1        |

I checked these points on the test images with straight lines and could see the source lines corresponding to the straight lane lines, and also see the lane lines become approximately parallel in the warped images.

Using the library function cv2.getPerspectiveTransform I then calculated the transformation matrix and the inverse transformation matrix. Using cv2.warpPerspective and the transformation matrix I get images looking like this:

![alt text][image6]

Here you see the original image with the source lines drawn in red and the warped version of the cropped image from above.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

For identification of the lane pixels I reused the code from the learning material for sliding windows search and search using previously known lines. This code is found in Part 1.5 of the the Jupyter Notebook file 'Project 4 Advanced Lane Finding.ipynp'. The sliding windows search starts by calculating a histogram of the lower half of and image. This typically looks like this:

![alt text][image7]

Based on this the code decides where to start the search for the left and right lanes. I modified the code to use a margin of 50 pixels and a minimum required number of pixels of also 50 in my final version of the sliding windows search. Part 1.5 of the notebook the code uses a minimum required number of pixels of 100. It produces results like this:

![alt text][image8]

The code use np.polyfit to identify a second order polynomial to the identified lane points. The yellow lines in the picure above shows the identified polynomials for the left and right lanes. The red pixels are the points identified for the left lane, the blues pixels are the points identified for the right lanes. Also the individual (sliding) windows are drawn as green boxes. You will noticed I used 10 windows instead of the 9 used in the course material. Not that this made any big difference.

From the course material I also have code which performs a search around known second order polynomials. This code can be used subsequently to a sliding windows search when left and right polynomials are already known. The advantage of this code is that it is less processing intensive. The code produce results like this:

![alt text][image9]

Here a search is conducted 50 pixels to either side of the already known polynomials. Again the resulting new polynomials are drawn in yellow, left points in red, right points in blue. The area searched is drawn in green.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In Part 1.6 of the the Jupyter Notebook file 'Project 4 Advanced Lane Finding.ipynp' there is code for calculation of the curvature and the offset from the lane center. This code is essentially a reuse of what was shown during the course material, and it use the following assumptions:

30/720 meters per pixel in y dimension.
3.7/700 meters per pixel in x dimension, assuming lane is about 700 pixels wide in the image.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In Part 1.7 and 1.8 I have code for warping the identified lines back on to the original image, as well as writing the calculated curvature and offset information. Essentially this is done using cv2.fillPoly,
cv2.warpPerspective with the inverse transformation matrix and cv2.addWeighted to draw the lane. The total result looks like this:

![alt text][image10]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Part 2 of the the Jupyter Notebook file 'Project 4 Advanced Lane Finding.ipynp' contains consolidating code so everything can be applied to a video stream. I wrote functions

* getSlidingWindowPolyFit(binary_warped)

* getPolyfitBasedOnPreviousFit(binary_warped, prev_left_fit, prev_right_fit)

both of which takes a binary thresholded and warped image as input and returns best guesses of left and right polynomials as well as a list of the identified lane points for the left and right lanes. Very important here is that both of these functions can return None if no polynomials are found, and both functions apply a sanity check on the identified polynomials. I experimented quite a lot with these sanity checks and ended up using distance between the 2 lines at y position 0 (upper edge of image) for discarding estimates.

I also applied (part of) the Line class which was proposed in the course and project material. I added code to add a new line fit and to calculate the average best fit based on the current list of fits held by the class. The main point is that the Line class keeps a list of up to the 24 newest line fits. New fits are appended to the list, and if there is more than 24 entries I remove the first element from the list, which is also the oldest fit.
The class also handles the entry of None for new fits, in which case it simply removes the oldest element again. In this case it will never empty the list but always keep the last identified fit in order to be able to propose some sort of lane line estimate. You could argue it would be better to give up if it has not been possible to identify any lane lines for the last 24 images in the video stream.

The last piece of code I would like to comment is the process_image image which is putting everything together. The function processes 1 image from the video stream and used a 2 global variables, namely the left and right lane instances of the Lane class to store the last lane estimates.

Here is a [link to my video result](./project_video.mp4)

I did not have time to try my code out on the 2 more challenging video's as I ran out of time for the project.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The main issues I faced was to tune the individual parts of the pipeline. This took quite a lot of time and was the reason I ran out of time in the end.

Calculating the binary thresholded version of the images is something which surely could be done differently that what I ended up doing. I presume my solution will have problems in more challenging cases with shades and different light conditions. However my solution is adequate for the project video.

Another aspect that took a lot of tuning was how to discard bad estimates. My solution produces right lane estimates which tend to swing to right of the actual lane at the top of the right lane estimate. In order to get rid of that problem I discard lane estimates where the distance between the left and right lanes at the top (y position 0 in the images) is too large. I identified 620 pixels as being too large by inspection. This does not really match the assumption of 700 pixels between lanes which is being used for the offset calculation, so possibly my offset calculation is not quite right.

In the end I did not apply any checks for my estimates lines being parallel. I had this check in the beginning, but found it to make no difference when I applied the distance check between the lanes at the top of the image (described above). In a real implementation checking for parallelism should surely be added as a further sanity check of the estimates.

A further measure to ret rid of the problem of bad right lanes was to discard estimates where signs of the second order part of the polynomials were different. This means that the lanes would have a different direction of their curves, and in my code this happened sometimes.

Lastly I had to choose the number of images over which to average the lanes. I ended up using 24, which corresponds to 1 second of the video. On the one hand this is quite a lot of images, on the other it led to much more "calmness" of the final lanes. The problem with using many fits for averaging is off course that the code becomes less responsive to quick direction changes of the lanes (and the car). I presume this might be a problem with the second challenge video which contains a road with many curves.

Lastly I did not use any averaging for the curvature and offset calculations. Instead I just display the results of the last image. A better implementation would have calculated averages for these values using the same window of estimates as I use for the lane fits.

All in all this project was a lot of fun, and one which I could spend a lot more time on, in order to make it better.
