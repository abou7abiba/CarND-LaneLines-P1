# **Finding Lane Lines on the Road** 


**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image_pipeline]: ./doc_images/Find_Lane_Pipeline.png "pipeline"
[image_region_calc]: ./doc_images/region_calculations.PNG "Region Calculations"
[image_line_slop]: ./doc_images/Line_slop.PNG "Line Slopes"

[image0]: ./test_images/solidYellowCurve2.jpg "Original Image"
[image1]: ./doc_images/solidYellowCurve2-Bluring-Config.jpg "Bluring Config"
[image2]: ./doc_images/solidYellowCurve2-Edge-Finder-Config.jpg "Edge Finder"
[image3]: ./doc_images/solidYellowCurve2-Region-Masked-dimensions.jpg "Region Mask"
[image4]: ./doc_images/solidYellowCurve2-Hough-Lines-Config.jpg "Hough Lines"
[image5]: ./doc_images/solidYellowCurve2-Image-Mix-Config.jpg "Mixed Image"

[//]: # (Link References)
[Identify-Region]: ./P1.ipynb#Identify-Region
[1]: https://docs.opencv.org/3.4/d6/d6e/group__imgproc__draw.html#ga311160e71d37e3b795324d097cb3a7dc
[2]: https://docs.opencv.org/3.4/d2/de8/group__core__array.html#ga60b4d04b251ba5eb1392c34425497e14
[3]: https://docs.opencv.org/3.4/dd/d1a/group__imgproc__feature.html#ga8618180a5948286384e3b7ca02f6feeb
[4]: https://medium.com/@maunesh/finding-the-right-parameters-for-your-computer-vision-algorithm-d55643b6f954
[5]: https://github.com/abou7abiba/opencv-gui-pipline-tuner

---
## **Pipeline Description**

My pipeline consists of 5 steps. Each step is applying specific processing to the input image and creting a processed image which is considered the input of the following step. The structure of the pipeline and sequance of steps are shown in the following diagram
![alt text][image_pipeline]

In the following I will explain the configuration of each of the steps.

### **Step 1: Bluring the Image**

In this step, the input is the original image, before any processing and we convert it to a grayscale  and then we apply the gaussian bluring using the `kernel_size`.

| Configuration | Input Image | Output  Image    |
| :---        |    :----:   |         :----: |
| `kernel_size=7`  | ![alt text][image0]       | ![alt text][image1]   |
|    | | |

### **Step 2: Edge Finder**

We use the Canny method to identify edges from the output of step 1. To do so we using two paramters `low_threshold` and `high_threshold`


| Configuration | Input Image | Output  Image    |
| :---        |    :----:   |         :----: |
| `low_threshold=58` `high_threshold=136`  | ![alt text][image1]       | ![alt text][image2]   |
|    | | |

### **Step 3: Mask Region of Intrest**

The region of interest is always relative to the original image. So based on the direction of the coordinates of the original image, we select a polygon dimension. However it is not that simple. I Identified 4 main variables to be able to calculate the polygon. These parameters are `x_up`, `x_bottom`, `y_up`, and `y_bottom` and these variables are explained in the diagram below relative to the original image.
![alt text][image_region_calc]

The values of these variables are **percentage** of the original hight and width. The diagram shows an image of width of `xsize` and hight `ysize` and showing a polygon `BADC` and the calculation of each point given the assumption that `AD = 2 * x_up` and `BC = 2 * x_bottom` and the polygon is always in the center of the width of the image at `xsize / 2`.

The diagram shows the calculation of `(x, y)` for each of the 4 points of the polygon. This is the way I used to identify and configure the region of interest.

Both `y_up` and `y_bottom` controls the hight of the maximum and the minimum of the region of interest which is giving me the ability to change the area of the interest and hence eliminate the points that are considered as noise.

I developed the method [`vertices ()`][Identify-Region] to convert the values of the maske region percentage to the absolute values relative to the input image. Then I use both the [`cv2.fillPoly()`][1] for regions selection and [`cv2.bitwise_and()`][2] to apply a mask to an image

| Configuration | Input Image | Output  Image    |
| :---        |    :----:   |         :----: |
|`x_up=0.06`, `x_bottom=0.42`, `y_up=0.37`, `y_bottom=0.02` | ![alt text][image2]       | ![alt text][image3]   |
|    | | |

### **Step 4: Identify Hough Lines**

The output of step 3 as masked region will be the input to apply the [`cv.HoughLinesP()`][3] so we can get the different segments of lines and draw it in yellow, and from it we extrapolate the left and right edges of the lane and draw it in red. I use `rho`, `theta`, `threshold`, `min_line_length`, `max_line_gap` as parameters for this method.

for this we use three methods `draw_lines()`, `draw_lane()` and `get_line()`.

#### **Description of `draw_lines()`**
In this method we draw all the line segments given to this method using a yellow color. The input for it is an array of `(x1, y1)` and `(x2, y2)` points as the start and end of each line.

#### **Description of `draw_lane()`**
This method is to draw the left and right boundaries of the lane, in red. To identify the boundaries of the lane (right edge and left edge), we use the slope of the line to get it as shown in the below diagram.

![alt text][image_line_slop]

The diagram shows that the left edge which is `AB` has a -ve slope while the right edge `DC` of the lane has a +ve slope. It also shows the slope of `EF` as its absolute value is < 1 while both `AB` and `CD` has a slop of absolute value > 1 which I tried to use it to remove the noise lines of segments.

The problem of this design is that each frame could have a noise different from the previous frame in the video. So to overcome this I created a new method `get_line()` to extrapolate the lane left and right boundaries

#### **Description of `get_line()`**
It is known that the line equation is

> `y = mx + c`

of first order where `m` represents the mean or slope of the line while `c` is a constant.

First I used the check `slope < 0` of each line to identify if it is for the left while `slope >= 0` for the right edges of the lane. The different input segments should be within the region masked in step 3 which is saved in the global variable `vertices_array`. 

I used the `np.polyfit(x, y, 1)` to extrapolate the line equation and get both `m` and `c` given all `x`  and `y` values of the line segments of this frame. However to overcome the noise in the images and keep a stable line over the different frames of the video I used 4 global buffers `m_list_l`, and  `m_list_r` to keep the **accepted** slop for both the left and right edges as well as `c_list_l`, and  `c_list_r` for the **accepted** constant value of the lines. From these values I got the mean value for `m` and `c` across the last set of frames based on the length of the buffer. I used a value of `buffer_len = 40` to get the mean across the last 40 frames.

In case of the test images, the `buffer_len = 1` as it is only for one image.

| Configuration | Input Image | Output  Image    |
| :---        |    :----:   |         :----: |
| `rho=2`, `theta=pi/180`, `threshold=7`, `min_line_length=4`, `max_line_gap=9` | ![alt text][image3]       | ![alt text][image4]   |
|    | | |

### **Step 5: Image Blender**
Finally we use the image with the drawn lines and lanes to be mixed with the original image to indicate what is the lane out of it. To do so we use parameters `α=0.9, β=1.0, γ=0.0`.

| Configuration | Input Image | Output  Image    |
| :---        |    :----:   |         :----: |
| `α=0.9, β=1.0, γ=0.0` | ![alt text][image4]       | ![alt text][image5]   |
|    | | |

---
## **Pipeline Configuration**

During the trip to get the best of the parameters, I went through different trials to do so. First I was using a different notebook to try the parameters. changing the parameters and execute the different cells of the notebook trying to get the best outcome. When I moved from testing the separate images to the video I found out that the parameters used for getting the images should not be identical for the videos. Then I cam across [this article][4] about finding the right parameters for the computer vision. I extended his work that you can find it in [my GitHub opencv-gui-pipline-tuner repository][5] to complete this work and get the right parameters for the images as well as for the videos.

Based on that I found out that I have three sets of configuration 

### **Test Images Configuration**

Here is the initial settings

```python
config = {'kernel_size': 9, 'low_threshold': 58, 'high_threshold': 136, \
        'x_up' : 0.06, 'x_bottom' : 0.42, 'y_up' : 0.37, 'y_bottom' : 0.02, \
        'rho' : 2, 'theta' : np.pi/180, 'threshold' : 7, 'min_line_length' : 4, 'max_line_gap' : 9}
```

### **Test Videos Configuration**

Then increasing the bluring buy increasing `kernel_size = 13` and changed the parameter for edge finder by decreasing both the `low_threshold = 37` and  the `high_threshold = 94`

```python
config = {'kernel_size': 13, 'low_threshold': 37, 'high_threshold': 94, \
              'x_up' : 0.09, 'x_bottom' : 0.42, 'y_up' : 0.36, 'y_bottom' : 0.0, \
             'rho' : 2, 'theta' : np.pi/180, 'threshold' : 7, 'min_line_length' : 4, 'max_line_gap' : 9}
```

### **Challenge Video Configuration**

To overcome the noise in the different parts of the videos with dark shadows, I changed the parameter for edge finder by increasing the `low_threshold = 52` and decrease the `high_threshold = 77` and modify the mask region to be raised a bit and modified the Hough Lines to raise the `threshold = 26`, `min_line_length = 1`, and increase `max_line_gap = 26`

```python
config = {'kernel_size': 13, 'low_threshold': 52, 'high_threshold': 77, \
              'x_up' : 0.14, 'x_bottom' : 0.42, 'y_up' : 0.33, 'y_bottom' : 0.08, \
             'rho' : 2, 'theta' : np.pi/180, 'threshold' : 26, 'min_line_length' : 1, 'max_line_gap' : 12}
```

---

## **Potential Shortcomings**

I have the following list of shortcomings

1. In the challenge video when the left yellow lane is not detected in some of the frames due to the bluring. I think the alternative was to transfer the yellow colors to white, but that needs an algorithm that i was not able to figure it out.
2. In some frames of the videos, there were vertical short lines which make some noise and I failed to find the configuration to remove them. I tried to use the |slope| > 1 as indicator but the results were not correct and the lane edge was always not correct.
3. Converting the video instead of processing it during playing the video to try to video annotation.
4. Using the pipeline steps as functions.
5. The hard coded configuration is not the right way of doing it. I designed the pipeline to get a dictionary of parameters for configuration, however it is still hard coded.

---

## **Possible Improvements**

Here are a list of possible improvements:

1. The pipeline showed be redesigned as a class of different pipelines (image, video, and folder) pipelines with classes of image processors with each one of it is applying a specific function (bluring, masking, Edge Detection, .. ). I applied that in my [pipeline tuning tool][5]
2. Using JSON files for configuring the pipeline. I applied that in my [pipeline tuning tool][5]
3. I used the CV video processing to annotate the videos with the lane lines which gave me the chance to enhance the configuration. I applied that in my [pipeline tuning tool][5]
4. Build a parameter tuning tool so we can tune the parameters when the around condition of the input video is changed so we can keep getting the best results. This could be automated and coupled with specific conditions like the time of the day if it is day or night or even the time in the year if it is a sunny or rainy day.
5. Using loggers for debugging and investigating the parameters (I applied that)
6. There is a need to change the colors for some areas of the image like the yellow lane so when we apply bluring it appears, we may need to process this in the image in a different way.

## **Rased Questions with no answers yet**

At this stage from the course and after this project, I had some questions that I didn't find answers for it yet.

1. What about identifying the lanes during driving the car by night with no clear vision? what will be the alternative?
2. What about driving in roads with no lanes, like driving in desert? I think it should need more detection for other types of obstacles.
3. I wonder how can I overcome these vertical lines? 
