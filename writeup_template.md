# **Finding Lane Lines on the Road** 

---


The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report



[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[image2]: ./test_images/results/images/gray.jpg
[image3]: ./test_images/results/images/blur_gray.jpg
[image4]: ./test_images/results/images/edge.jpg
[image5]: ./test_images/results/images/hough_lines.jpg
[image6]: ./test_images/results/images/masked.jpg
[image7]: ./test_images/results/solidWhiteCurve.jpg

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

This pipeline consists of 5 steps. 

In step 1 we take this image and get rid of all the colors. This results in a grayscaled image with only one dimension for color. 
![gray scaled image][image2]
In step 2, we added some blur to it in order to not distract our algorithm from any minute details. 
![gray blur image][image3]
At this point we apply canny edge detection in order to see those regions where the derivative for intensity is high. This picks up all the shapes in the image.
![canny edge image][image4]
We now focus our attention to the region in front of the car. In order to do this we mask the area where we want canny edge result.
![masked image][image6]
After that we use hough transform to find lines in this image. Hough tranform function input takes parameters to fine tune output value.

``` 
hough_lines(img, rho, theta, threshold, min_line_len, max_line_gap)
```

![hough lines][image5]

Output of hough lines is the input to our drawlines function. There are multiple lines on either side of the point of view. We take the input array and calculate the slope. We further add some threshold to filter out unwanted noise created by some small line segments which are not of the road pointing forward.

```
for line in lines:
        for x1,y1,x2,y2 in line:
            if (x2-x1 != 0):
                if (y1>y_min and y2>y_min):
                    m = (y2-y1)/(x2-x1)
                    if (m > -0.8 and m < -0.2):
                        LEFT_SIDE_OF_ROAD
                    elif (m < 0.8 and m > 0.2) :
                        RIGHT_SIDE_OF_ROAD
```

At this point I am doing 2 things. Keeping a track of the longest line in each direction to get one reference point of a line segment and averaging all slopes together to get average slope value.

I am breaking a line in each direction into 3 parts. One consists of the longest segment that we just found and other two are above and below that segment, ie top and bottom of the masked area.

```
 intercept_left = longest_left_lane_segment[2] - slope_left_val*longest_left_lane_segment[0]
 intercept_right = longest_right_lane_segment[2] - slope_right_val*longest_right_lane_segment[0]
```

Using these values I am drawing the lines in each direction. 

![final image][image7]




### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming in this approach is the longest segment keeps changing as the vehicle moves in a video frame. So the longest line causes lot of jitter and the overlay is not smooth.

Another shortcoming is when there is a curve on the road this line cannot be accurately drawn. Also if another vehicle is in the front of this car, it will totally interfere with the line segments used to interpolate these values.


### 3. Suggest possible improvements to your pipeline

I see some folks applying linear regression on either side of the line segments, that could potentially make the lines appear more deterministically. 

Another potential improvement would be to not rely on max of line segments to find top corners of the lanes. Rather narrow down on the lane trajectory and only use values inside of that region.
