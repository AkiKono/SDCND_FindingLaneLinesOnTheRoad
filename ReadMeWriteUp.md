#**Finding Lane Lines on the Road** 

##Writeup Template

###You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

The goals / steps of the written report
* Describe the pipeline
* Identify any shortcomings
* Suggest possible improvements

[//]: # (Image References)

[image1]: ./Test_images/whiteCarLaneSwitch.jpgresult.jpg "Result"
[image2]: ./Test_images/whiteCarLaneSwitch.jpgyellow.jpg "YellowMask"
[image3]: ./Test_images/whiteCarLaneSwitch.jpghighlight.jpg "Highlight"
[image4]: ./Test_images/whiteCarLaneSwitch.jpggray.jpg "Grayscale"
[image5]: ./Test_images/whiteCarLaneSwitch.jpgblur_gray.jpg "BlurGrayscale"
[image6]: ./Test_images/whiteCarLaneSwitch.jpgedges.jpg "Edges"
[image7]: ./Test_images/whiteCarLaneSwitch.jpgregi_of_int.jpg "RegionOfInterest"
[image8]: ./Test_images/whiteCarLaneSwitch.jpgmasked_edges.jpg "MaskedEdges"
[image9]: ./Test_images/whiteCarLaneSwitch.jpg "MaskedEdges"
[image10]: ./Test_images/whiteCarLaneSwitchLines.jpg "HoughTransformLines"
[image11]: ./Test_images/extra5850msimage.jpg "failure"


---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 7 steps. () are the openCV functions that are used.
* mask yellow color pixels. (`cv2.inRange`)
* convert yellow color pixels to white color pixels. (`cv2.addWeighted`)
* convert to grayscale. (`cv2.cvtColor`)
* apply Gaussian Blur. (`cv2.GaussianBlur`)
* apply Canny Edge Detection. (`cv2.Canny`)
* mask out outside of region of interest. (`cv2.fillPoly` & `cv2.bitwise_and`)
* apply Hough Transform to find straight lane lines. (`cv2.HoughLinesP`)

As an example, start from this law image.
![alt text][image9]

I am interested in a solid yellow lane line on the left and a segmented white lane line on the right. To make both lines equally detectable, yellow colored pixels is converted to white colored pixels.

To do so, the first step is to apply yellow color mask using `cv2.inRange`. Since the pure yellow color value is [255,255,0] in RGB, the range is set by the lower boundary: [180,180,0] and the upper boundary: [255,255,150]
![alt text][image2]

Then, overwrite it on the original image using `cv2.addWeighted`.
![alt text][image3]

Next, to find the contour of the lane lines, Canny Edge Detection is going to be used. Before that, however, the colored image is simplified by converting it to grayscale using `cv2.cvtColor`.
![alt text][image4]

Another preparation before applying Canny Edge Detection is a Gaussian blur (`cv2.GaussianBlur`) to reduce noise and small details on the image. Kernel size is set to 5.
![alt text][image5]

Now, the image is ready for Canny Edge Detection (`cv2.Canny`). It detects edges by intensity gradients within a given low and high threshold. They are set to 75 for low and 180 for high where 0 is the lowest and 255 is the highest.  
![alt text][image6]

In order to find lane lines from these random edges, a region of interest is defined as shown below.
![alt text][image7]

Then, edges outside the region is masked out, using `cv2.fillPoly` and `cv2.bitwise_and`.
![alt text][image8]

Then, Hough Transform (`cv2.HoughLinesP`) is applied to find straight lines. Grid accuracy in the polar Hough space is set to be rho = 5 pixels and theta = 1 degree. The maximum vote for a grid to be considered as a line is set to 100. Minimum length of the line is 80 pixels and maximum allowable line gap is set to 60 pixels. The Hough Transform found the lines shown in the below image.
![alt text][image10]

I would like to draw a single unsegmented line for left and right lanes. So first, slope and y-axis intersection of each line are calculated using the equations shown below

`y = mx + b`
* `m = (y2-y1)/(x2-x1)` <br />
* `b = y1-m*x1`

Where `m = slope`, `b = y-axis intersection`, `(x1,y1)(x2,y2) = end points of a line`. <br />
Next, group them by negative slope lines and positive slope lines. Negative slope lines represent left lane, and positive slope lines represent right lane. Then, take median values for each slope and y-axis intersection to draw a single line for left and right lanes from bottom of the image to the apex of the region of interest. The final result is shown below.
![alt text][image1]


###2. Identify potential shortcomings with your current pipeline

For some images in the challenge.mp4 movie, my pipeline does not work as shown below.
![alt text][image11]

My pipeline could not find lane lines due to the shadow of the trees and the color difference on the road.

In the same way, it may not work when there are faded lane lines, unnecessary old lanes remained on the road, or cars in front occupying some portion of the lane lines.

Also, weather such as rain, sand storm, snow, and fog, also affect my pipeline results. The use of wiper during such weather condition might affect the results if the camera is place inside the windshield.

Poor lighting conditions such as head lights from oncoming cars during the night, darkness due to no street lights, backlight evening, or driving inside a tunnel definitely affect my pipeline results. Especially Canny Edge Detection may not work properly since the intensity of the lane lines mark could be so different from ones from during daytime.

Another shortcoming could be that it only tries to detect straight lines. Curves are not included. Also, it is possible that lane lines are out of Region of Interests when the lane lines are curved.

The most importantly, it takes a lot of time to tune the parameters. Color mask RGB value ranges, Canny Edge Detection thresholds, Hough Transform parameters, all of them were fine tuned for given particular test images only. For a practical application, however, it has to detect lane lines in real time in any condition. This approach simply takes too much time and requires constant parameter tuning while driving. Thus, we need a faster and more robust automatic lane lines detection system.

###3. Suggest possible improvements to your pipeline

Deep learning is necessary for a faster and more robust automatic lane lines detection, and I believe that is why we are going to learn it next.

Besides deep learning, however, a possible improvement on my pipeline would be to apply lane tracking. Since lane lines on the roads are most often continuous, lane lines can be tracked in consecutive video frames. By filtering out end points of lines detected by Hough Transform that are far from the previous lane lines location, my pipeline can be improved

Another easy possible improvement could be to delete lines with slopes that are close to zero in the draw_lines function.

Another possible improvement would be to use lane width. In the United States, U.S. Department of Transportation specifies lane width for the different type of roadways.

| Type of Roadway | Rural (feet)| Urban (feet)|   
|---|---|---|
| Freeway | 12 | 12 |   
| Ramps (1-lane) | 12-30 | 12-30 |   
| Arterial  | 11-12 | 10-12 |  
| Collector | 10-12 | 10-12 |
| Local | 9-12 | 9-12 |
(https://safety.fhwa.dot.gov/geometric/pubs/mitigationstrategies/chapter3/3_lanewidth.cfm)

For freeway, the lane width is required to be 12 feet and constant, which means the spacing between detected left and right lane lines at the bottom of the image should also be nearly constant. By filtering out lane line end points at the bottom of the image that are far apart from an expected range, my pipeline can possibly be improved. Or, simply add lane width checking after detecting the lane lines.

Another potential improvement could be to use HLS color mask. I used RGB color code to find yellow color. However, HLS color code can detect light color, like white and yellow, more easily because it parameterize colors by lightness.

Another useful technique to find lane lines is Bird's-eye view mapping. Warp Perspective Mapping (WPM) and Inverse Perspective Mapping (IPM) are such examples. I need the height of the fixed camera from the ground, and its angle. Both techniques assume grounds to be flat.  (http://www.vision.caltech.edu/malaa/publications/aly08realtime.pdf)
(https://pdfs.semanticscholar.org/d082/40c17eb12bdc7f6a060781d624d2f6a2f3d4.pdf)

Another robust algorithm to detect lane lines is RANdom Sample Consensus (RANSAC). According to the paper, it works even when there is only one lane line on the road.
(http://ieeexplore.ieee.org/document/7244831/?reload=true)

For detecting curves, hyperbola model can be used to find curved lane lines. The author of the paper _"A multi-step curved lane detection algorithm based on hyperbola-pair model"_ states that "Experiment results show a robust performance in a noise condition, such as part of the lane markings are occupied by other vehicles in the lanes."
(http://ieeexplore.ieee.org/document/5585398/)

By applying above improvements, my pipeline would be more robust and reliable.
