
# **Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./image1_examples.jpg
[image2]: ./image2_examples_color_hist.jpg
[image3]: ./image3_examples_spatial.jpg
[image4]: ./image4_examples_hog.jpg
[image5]: ./image5_examples_boxes.jpg
[image6]: ./image6_examples_windows_result.jpg
[image7]: ./image7_examples_heatmap.jpg
[video1]: ./project_video.mp4

### [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
#### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

##### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

##### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the code cell #4 of the IPython notebook. I used implementation from the course lessons. I included hog features and also spatial binning and color histogram.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like. I did the same for spatial binning and color histogram.
Example of color histogram of images given above:

![alt text][image2]

Example of spatial binning of images given above:

![alt text][image3]

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image4]

##### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and finally I'v chosen following parameters for each of features:

| Param | Value |
| ----- | ----- |
| color_space | 'YCrCb' |
| orient | 9 | 
| pix_per_cell | 8 |
| cell_per_block | 2 |
| hog_channel | "ALL" |
| spatial_size | (16, 16) | 
| hist_bins |  16 |

Different combinations of parameters resulted in either big increase of features (so more time to compute) without significant increase in training results of even in decrease of training results.

##### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using all the features, to get the most of the information. I tried selecting out some features in quizes in course lessons, but it didn't give me results significant enough to apply it in my project.
The code for training the SVM is in cell #8 in the notebook.

#### Sliding Window Search

##### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to search random window positions at random scales all over the image and came up with this (ok just kidding I didn't actually ;):
I used the sliding window search provided in one of the last course lessons on the project topic - the one with hog map for entire image. I tested both, this and the sliding window search that was introduced first, but my testing just comfirmed what was said in the course - the latter way gave better results in shorter amount of time. I only removed the line with dividing pixel values to range 0-1 as I used opencv to read .png images to train the classifier on, so the classifier was training on images in range 0-255 (actually this single line gave me a lot of pain when testing, because while focusing on how does it work, I didn't notice that).
The code may be found in cell #10 in a notebook.

Ultimately I searched with three scales:
* scale 1.5 within for cars far away 
* scale 2.0 within for cars in the middle
* scale 2.5 for cars close
all using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.

![alt text][image5]

##### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

  Here is example image:

![alt text][image6]
---

### Video Implementation

##### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


##### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected. 
I also implemented class _Detection_ to track previous detections. For each new frame, I was analysing previous detections. I was inspecting all detections that were somewhat in the middle of the picture - so they couldn't appear out of nowhere. If I found a detection in previous frame nearby, to the right-lower direction from a current one, that meant to me that the same car was detected there so I added heat to the heatmap for this detection. However, if there was no detection in previous frames, I removed the detection, assuming that it's a false positive. Looking at the improvement in video result after applying this method, I concluded this method worked quite well.

Here's an example result showing the heatmap from a test frame:
![alt text][image7]



---

#### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The most problematic was the issue with false positives. I spent most of the time and developing checking for this. What could work better, would maybe be trying to detect all lane lines and check if found object is linearly moving forward.

