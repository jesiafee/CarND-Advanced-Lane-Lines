## Writeup:P2 Advanced Lane Finding

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

## Key Points
Here are the key points what I will do as follows:
* Calibrate camera;
* Applying algorithm on single iamges;
* Applyting algorithm on project videos.

---

### Camera Calibration

#### 1. I import the images in the folder *camera_cal*, and create empty arrays to store the information of image points and object points.
I import the test images by glob, which allows reading in images with a consistent file name.
The empty arrays to store image points and object points refer to the method in the course.

#### 2. I obtain the image points by function findChessboardCorners(), and add them into the arrays created above; the object points array is filled with the data created by ndarrays.
The image points are obtained from the test images in folder camera_cal, using the cv2.findChessboardCorners(); the object points are created by the method mentioned in the course.

#### 3. I calibrate the camera by the arrays got above, and get the undistort images.
The undistort image can be output by applying the cv2.calibrateCamera() and cv2.undistort(). And the parameters output in the process, can be used to calibrate other images captured by the same camera.

![calibrate camera](https://user-images.githubusercontent.com/41224032/43309023-1d0d49fa-91b6-11e8-9055-ab98ff8b9ef5.png)


### Pipeline (single images)

#### 1. Get undistort iamges by applying the matrix got above.

![undistort images real](https://user-images.githubusercontent.com/41224032/43310027-055f3b94-91b9-11e8-8a42-bf8af0cc8476.png)

#### 2. Give the undistort image a perspective transform.
The straight_line images own roughly parallel lines, so I adjust the quadrangle many times to get an approximate perspective transform.

```python
offset = 100
src = np.float32([[image_width*0.43,image_height*0.65],
                  [image_width*0.57,image_height*0.65],
                  [image_width*0.19,image_height*0.88],
                  [image_width*0.81,image_height*0.88]])
dst = np.float32([[offset,offset],
                  [image_width-offset, offset],
                  [offset,image_height-offset],
                  [image_width-offset,image_height-offset]])
```
Then I get the unwarped image by using cv2.getPerspectiveTransform() and cv2.warpPerspective() altogether.

![perspective transform](https://user-images.githubusercontent.com/41224032/43310080-2f6506c6-91b9-11e8-83e0-409ce32e9f81.png)

#### 3. process the unwarped image.
I tried a few process methods.

![processed unwarp](https://user-images.githubusercontent.com/41224032/43310254-ab93cd2c-91b9-11e8-818a-fcd869f98b02.png)

It seems that the 1st,2nd,3rd,6th do the best, so I combined them as input.

![combined](https://user-images.githubusercontent.com/41224032/43310442-22b22570-91ba-11e8-9c37-4ffb6141fe65.png)

Next, calculate its histogram peaks of the bottomhalf(3/5), and display it on the image.

![histogram](https://user-images.githubusercontent.com/41224032/43310564-7c71d90c-91ba-11e8-8f45-edbb610c6aba.png)

Then I polyfit the lane lines by yellow lines, and mark the red and blue on it.

![marked unwarped](https://user-images.githubusercontent.com/41224032/43310733-0063045c-91bb-11e8-896b-73394838d15e.png)

#### 4. mark lane lines and the estimated curvature on the undistort images.
Since I mark the lane lines and polynomial lines on the image, I warp it back on the undistort iamge.

![add mark back](https://user-images.githubusercontent.com/41224032/43310879-689ce632-91bb-11e8-97ae-91534a605b41.png)

Then I calculate the estimated curvature by average the left and right curvature, and add the result on the undistort image.

![add words on](https://user-images.githubusercontent.com/41224032/43310972-b28b635e-91bb-11e8-913b-bc07430c2a50.png)

Finally, I add the green mask on the distorted image, as a output result.

![result](https://user-images.githubusercontent.com/41224032/43311184-3d691b42-91bc-11e8-8c24-fbaa9207f6ee.png)

---

### Pipeline (video)

#### output the corresponding images frame by frame, and output the video.
Well, all the process are mentioned above, the only difference is that the output funcion, which is referred to the codes mentioned in the *P1 Finding Lane Lines*.
```python
test_output = 'project_video_test.mp4'
clip1 = VideoFileClip("project_video.mp4")
white_clip = clip1.fl_image(process_image) #.subclip(46,47) #NOTE: this function expects color images!!
%time white_clip.write_videofile(test_output, audio=False)
```
I output the result and store it in the root folder(project_video_test).
Done.

---

### Discussion

#### Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

##### 1.The main problem is that vertical lines disturbance. In the output video(46s~47s), the green-masked area shake almost 3 frames. After analysing it, the reason is just the vertical lines disturbance.
How I make it more robust... The effective method is to create an individual list to store information like left-right base points, the calculated curvature. When the base points jump obviously, for example, (|left base point - right base point| / 3), I can ignore the jump and adopt the previous points. So I can avoid it probably.

##### 2.The estimated curvatures are unbelievable...
Maybe it's related to the accuracy of the perspective transformation. I can't do more to get more accurate results.

##### 3.It takes a long time to process the video in the workspace(11 minutes), and more than double time on local machine(30 minutes).
I get the polynomial by the method mentioned in the course using sliding window, which will take up much time. Maybe I can apply a new function to get the first frame of the video, then I use a more effective function mentioned in the course search from prior(avoid blind searches). This may increase its efficiency.

##### 4.The algorithm fails to process the challenge and harder challenge video, the green-masked area jumped drastically. 
It's dramatically important to create a new structure to store historical data. When data with good consistency jump drastically, the previous data should be used instead.
