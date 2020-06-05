# Finding-Lane-Lines

[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

---
![sdc-1-1024x427](https://user-images.githubusercontent.com/25509152/33837471-770be3a2-de9d-11e7-97cc-50c6224bfcc3.png)

---

[//]: # (Image References)
[image1]: ./test_images_output/origin.jpg "origin"
[image2]: ./test_images_output/gray.jpg "gray"
[image3]: ./test_images_output/blur.jpg "blur"
[image4]: ./test_images_output/edges.jpg "edges"
[image5]: ./test_images_output/roi.jpg "roi"
[image6]: ./test_images_output/lines.jpg "lines"
[image7]: ./test_images_output/result.jpg "result"

---


### Table of Contents


- Overview
- The project
- Dependencies
- Ideas for Lane Detection Pipeline
- Build a Lane Finding Pipeline
- Test on videos
- Improved lane finding pipeline
- Reflection

---

## Overview :point_left:

In this project, I will use the tools about computer version to identify lane lines on the road.

I will develop a pipeline on a series of individual images, and later apply the result to a video stream

---

## The project :ear:

The goals / steps of this project are the following:

* Make a pipeline that finds lane lines on the road
* Improve this pipeline
* Reflect on my work



---




## Dependencies

This lab requires docker images:

* [CarND Term1 Starter Kit](https://github.com/udacity/CarND-Term1-Starter-Kit)




---
## Ideas for Lane Detection Pipeline


Some OpenCV functions I have used for this project are:

1- cv2.inRange() for color selection  
2- cv2.fillPoly() for regions selection  
3- cv2.line() to draw lines on an image given endpoints  
4- cv2.addWeighted() to coadd / overlay two images cv2.cvtColor() to grayscale or change color 
5- cv2.imwrite() to output images to file  
6- cv2.bitwise_and() to apply a mask to an image  


---

## Build a Lane Finding Pipeline

Using the following image as example, my pipeline consisted of 6 steps. 
![alt text][image1]

1. Converted the images to grayscale from RGB model 
![alt text][image2]  

2. Use cv2.GaussianBlur() to blur the image
![alt text][image3]  

3. The first core operation: detect edges of a gray model image
![alt text][image4]  

4. After the edges have been got, my next step is to define a region of interest(i.e., ROI), this method is old but efficient. Cause the camere installed on the car is fixed, so the lane lines is in a specific region, usually a trapezoid.
![alt text][image5]  

5. Anothe core operation: hough transform edges to a set of lines represent by start point and end point. Hough transform get the image of lane lines we want.
![alt text][image6]  

6. Finally, we add the lane lines image and innitial image together.
![alt text][image7]  


---
## Test on videos

You know what's cooler than drawing lanes over images? Drawing lanes over video!

We can test our solution on two provided videos:

`solidWhiteRight.mp4`

`solidYellowLeft.mp4`  

The result is in [test_videos_output](https://github.com/liferlisiqi/Finding-Lane-Lines/tree/master/test_videos_output): white.mp4 and yelloe.mp4.


---
## Improved lane finding pipeline

At this point, if you were successful with making the pipeline and tuning parameters, you probably have the Hough line segments drawn 

onto the road, but what about identifying the full extent of the lane and marking it clearly as in the example video (P1_example.mp4)?

Think about defining a line to run the full length of the visible lane based on the line segments you identified with the Hough 

Transform. As mentioned previously, try to average and/or extrapolate the line segments you've detected to map out the full extent of

the lane lines. You can see an example of the result you're going for in the video "P1_example.mp4".

Go back and modify your draw_lines function accordingly and try re-running your pipeline. The new output should draw a single, solid 

line over the left lane line and a single, solid line over the right lane line. The lines should start from the bottom of the image and 

extend out to the top of the region of interest.


---
## Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps. First, I converted the images to grayscale, then I .... 

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by ...

If you'd like to include images to show how the pipeline works, here is how to include an image: 

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...


---
[Back To The Top](#README.md) :point_up:

---

End :raising_hand:
