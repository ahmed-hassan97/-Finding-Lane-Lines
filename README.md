# Finding-Lane-Lines using python and opencv

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

## hyperparameters

```

      # Tunable parameters (I will use values from quizzes)
      kernel_size = 5       # Gaussian blur kernel size
      low_threshold = 40    # Canny low threshold for gradient value
      high_threshold = 130  # Canny high threshold for gradient value
      rho = 1               # distance resolution in pixels of the Hough grid
      theta = np.pi/90      # angular resolution in radians of the Hough grid
      threshold = 30        # minimum number of votes (intersections in Hough grid cell)
      min_line_length = 30 # minimum number of pixels making up a line
      max_line_gap = 300    # maximum gap in pixels between connectable line segments

      # My region of interest will be a centered trapezoid rising from the bottom of the image.
      # These parameters tune the shape of the trapezoid.
      roi_horiz_top = 0.45 # x-coord of trapezoid's top left point will be img.shape[1]*roi_horiz_top
                          # x-coord of trapezoid's top right point will be img.shape[1]*(1.-roi_horiz_top)
      roi_horiz_bot = 0.05 # x-coord of trapezoid's bottom left point will be img.shape[1]*roi_horiz_bot
                          # x-coord of trapezoid's bottom right point will be img.shape[1]*)(1.-roi_horiz_bot)
      roi_vert = 0.6      # y-coord of trapezoid's top will be imshape[1]*roi_vert_frac

      # Parameters (channel value ratios) used to filter out everything that is far from white or yellow.  
      # From a quick plt.plot(test_image[400,:,:]), it's possible to discover the approximate range
      # of red/green and red/blue ratios that correspond to the yellow lane line: 
      yellow_g2r_low = 0.75 
      yellow_g2r_high = 1.1
      yellow_b2r_low = 0.0
      yellow_b2r_high = 0.75
      # ...and the white lane lines:
      white_g2r_low = 0.95
      white_g2r_high = 1.05
      white_b2r_low = 0.95
      white_b2r_high = 1.05

```



---

## Build a Lane Finding Pipeline

Using the following image as example, my pipeline consisted of 6 steps. 
![alt text][image1]

1. Converted the images to grayscale from RGB model 
![alt text][image2]  
```
        def grayscale(img):
        """Applies the Grayscale transform
        This will return an image with only one color channel
        but NOTE: to see the returned image as grayscale
        (assuming your grayscaled image is called 'gray')
        you should call plt.imshow(gray, cmap='gray')"""
        return cv2.cvtColor(img, cv2.COLOR_RGB2GRAY)
   
```
2. Use cv2.GaussianBlur() to blur the image
![alt text][image3]  

```
      def gaussian_blur(img, kernel_size):
      """Applies a Gaussian Noise kernel"""
      return cv2.GaussianBlur(img, (kernel_size, kernel_size), 0)

```

3. The first core operation: detect edges of a gray model image
![alt text][image4]  

```
      def canny(img, low_threshold, high_threshold):
       """Applies the Canny transform"""
      return cv2.Canny(img, low_threshold, high_threshold)


```

4. After the edges have been got, my next step is to define a region of interest(i.e., ROI), this method is old but efficient. Cause the camere installed on the car is fixed, so the lane lines is in a specific region, usually a trapezoid.
![alt text][image5]  

```
     def region_of_interest(img, vertices):
      """
      Applies an image mask.

      Only keeps the region of the image defined by the polygon
      formed from `vertices`. The rest of the image is set to black.
      `vertices` should be a numpy array of integer points.
      """
      #defining a blank mask to start with
      mask = np.zeros_like(img)   

      #defining a 3 channel or 1 channel color to fill the mask with depending on the input image
      if len(img.shape) > 2:
          channel_count = img.shape[2]  # i.e. 3 or 4 depending on your image
          ignore_mask_color = (255,) * channel_count
      else:
          ignore_mask_color = 255

      #filling pixels inside the polygon defined by "vertices" with the fill color    
      cv2.fillPoly(mask, vertices, ignore_mask_color)

      #returning the image only where mask pixels are nonzero
      masked_image = cv2.bitwise_and(img, mask)
      return masked_image


```

5. Anothe core operation: hough transform edges to a set of lines represent by start point and end point. Hough transform get the image of lane lines we want.
![alt text][image6]  

```

    def hough_lines(img, rho, theta, threshold, min_line_len, max_line_gap):
    """
    `img` should be the output of a Canny transform.
        
    Returns an image with hough lines drawn.
    """
    lines = cv2.HoughLinesP(img, rho, theta, threshold, np.array([]), minLineLength=min_line_len, maxLineGap=max_line_gap)
    line_img = np.zeros((img.shape[0], img.shape[1], 3), dtype=np.uint8)
    draw_lines(line_img, lines)
    return line_img
    
```

6. Finally, we add the lane lines image and innitial image together.
![alt text][image7]

```
    def weighted_img(img, initial_img, α=0.8, β=1., γ=0.):
    """
    `img` is the output of the hough_lines(), An image with lines drawn on it.
    Should be a blank image (all black) with lines drawn on it.
    
    `initial_img` should be the image before any processing.
    
    The result image is computed as follows:
    
    initial_img * α + img * β + γ
    NOTE: initial_img and img must be the same shape!
    """
    return cv2.addWeighted(initial_img, α, img, β, γ)
    
```

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
