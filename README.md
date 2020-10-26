# **Finding Lane Lines on the Road** 
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

<img src="examples/laneLines_thirdPass.jpg" width="480" alt="Combined Image" />

Overview
---

When we drive, we use our eyes to decide where to go.  The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle.  Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.

In this project I developed a pipeline and I configured it to be able to apply it on the following:
- Images in the [test_images](./test_images) folder. The output of the annotated images are in the [test_images_output](./test_images_output) folder
- Videos in the [test_videos](./test_images) folder. The output of the annotated videos are in the [test_videos_output](./test_videos_output) folder

Project Code
---
The submitted code includes 
1. The [project notebook](./P1.ipynb)
2. An [HTML export](P1.html) of the project notebook
3. Another [notebook](./parameters_test.ipynb) that I used to test the different parameters to identify the best configuration. Later, I used my [pipeline tuning tool][1] to get the best configuration.
4. A [log file](./P1_info_out.log) that includes the log of the different runs for the pipelines steps across the multiple trials.

Writeup
---

The submitted [writeup documentation](./writeup.md) contains the following sections:

- **Pipeline Description** which contains the description and the diagrams that explains the sequence of steps used to build the pipeline
- **Pipeline Configuration** which explains the changes in the configuration used from the images to the videos till the challenge video.
- **Potential Shortcomings** that I identified during the implementation and testing of the project like the changes in the design of the pipeline, the problem with the vertical lines, and the noise ines due to the shadows in the challenge video.
- **Possible Improvements** I described some of the improvements and I applied that in my [pipeline tuning tool][1] 
- **Rased Questions with no answers yet** which I kept asking myself and I didn't find an answer for it yet at this stage of the course. For example what if the road has no lanes, how can we overcome that. Like when driving in the desert?

[//]: # (Link References)

[1]: https://github.com/abou7abiba/opencv-gui-pipline-tuner