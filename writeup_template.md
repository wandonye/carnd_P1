# **Finding Lane Lines on the Road**

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 6 steps:
1. I converted the images to grayscale
2. Smoothed the grayscale image
3. Compute edge of the smoothed image with Canny algo
4. Mask the edge image with interested region
5. Fit line segments to points in the masked edge image
6. Draw a single line on the left and right lanes

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by first computing the slope angle of each line segments (with atan2). Line segments with slope that is too flat was removed. The remaining lines were grouped with respect to the slopes. When difference of the angles of two lines is less than 2 degree, the two lines are considered in the same group(connected component). For each group, I constructed a line by the following steps:
1. average all the points (two for each line segments) within this group, and get one point P
2. average the angles of all the line segments within this group and get an angle alpha
Now the line represent this group is the line passing P with angle alpha.

There are still possibly more than two groups(connected components) and thus more than two consolidated lines. I split the consolidated lines into two lists depends on their slopes being bigger thant pi/2 or not. Then rank each list with respect to the total length of line segments within each group(connected component). Finally I picked the top one in each list with the largest total length. This means a majority of line segments lie on these two lines.



### 2. Identify potential shortcomings with your current pipeline
The above approach clusters line segments only with respect to their angles. When there is another parallel segment far away from the line segments in the same group, the average of this group will be shifted. This is very likely to happen when driving in the left most or right most lane and the shoulder of road may contribute such kind of lines. So I created another `draw_lines` function. In this function, I did the grouping with both the angle and rho (distance from the line to the origin). So we are essentially doing clustering in the Hough space. This step can be merged with `cv2.HoughLinesP` to improve performance.

Another problem with this pipeline is that when certain frame has low quality image due to shadow, broken road, missing lane mark and so on, the lane lines cannot be computed correctly. But since the lines are almost always changing smoothly, it is possible to infer missing lanes or correct errors by extrapolating. (Interpolating is not allowed since in practice we don't have the lines for the next moment) So I wrote another pipeline.

The new pipeline consisted of 7 steps:
1. I converted the images to grayscale
2. Smoothed the grayscale image
3. Compute edge of the smoothed image with Canny algo
4. Mask the edge image with interested region
5. Fit line segments to points in the masked edge image
6. Fit a single line on the left and right lanes
7. Compare left line and right line from 6 with these in the previous frame, if the difference is too big, then deny the new lines and use these from the previous frame.

A issue of the new pipeline is that when the road curves and denying in step2 occurs for some reason, the lines may freeze at an earlier frame and deviate more along the curve. But this can be easily solved with more advanced denoise methods. Non-local means filter is a good candidate for such kind of noise(similar to salt-and-pepper noise). Unlike averaging or Gaussian filter, which will deviate from the true line because the averaging effect, Non-local means can remove them without shifting away.

Another potential issues is when the vehicle is changing lane and the road is curvy, the splitting condition of left lane and right lane may fail.


### 3. Suggest possible improvements to your pipeline

Further improvements addressing the last issue mentioned above can be a modified Hough transformation, which may fit polynomials to edges instead of lines.
