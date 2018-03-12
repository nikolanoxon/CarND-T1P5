## Writeup

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/figure_14.png
[image6]: ./examples/figure_15.png
[image7]: ./examples/figure_16.png
[image8]: ./examples/figure_17.png
[image9]: ./examples/figure_18.png
[image10]: ./examples/figure_19.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in code cell 3 of `CarND-Vehicle-Detection.ipynb`.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YUV` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and color spaces before I identified a satisfactory result. This was not just based on the accuracy of the classifier, but also on the amount of false positives and true positives in the project video. The largest improvement I found was in the selection of the YUV color space. This reduced the false positives to a very low value in my test frames while retaining the detection rate used in other color spaces.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using HOG features, color bins, and color histograms extracted from the 3 channels in the YUV color space. The code for this can be found in the 6th code window of `CarND-Vehicle-Detection.ipynb`.

I used the GTI and KITTI datasets for my vehicle and non-vehicles. The data was first shuffled, then the feature were extracted and normalized. I used a ratio of 80% training data to 20% validation data. The test accuracy of my SVC was 98.9%, which I felt was sufficcient for this project since I was easily able to filter out false positives using methods described below.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

My sliding window implementation can be found in code cell 3 of `CarND-Vehicle-Detection.ipynb`. After some experimentation, I settled on having four bands of windows which looked at the near, near-middle, far-middle, and far portions of the road. For example, the near band has a y-range from 400 to 628 pixels (and a scale of 2.0), while the far band has a range of 400 to 464 pixels (and a scale of (1.0). This captures almost the entire road from the horizon to the front of the vehicle.

Initially I included more bands, but found that my performance and speed improved when I reduced down to 4 bands. I used an overlap of 80%, which was likely overkill for this project, but did not seem to have any negative impact. If I were more interested in speed performance, I might reduce this down to 50%.

![alt text][image3]

Note: In the above image, the overlap is set to 50% for readability.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on four scales using YUV 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

Here's a [link to my video result](./project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.

Every frame that an object is detected, if it is within a minimum distance from an existing target, the new coordinates are added to a moving average for the vehicle. This moving average consists of the last seven frames. If a vehicle is not detected for 3 frames, the target is deleted. I also added a minimum target lifetime of 3 frames before the vehicle is considered real. This created very smooth bounding boxes with no false positives for the project video.

Here's an example result showing the heatmap from a series of frames of video and the bounding boxes:

![alt text][image5]
![alt text][image6]
![alt text][image7]
![alt text][image8]
![alt text][image9]
![alt text][image10]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The largest issues I encountered were sizing the bounding boxes correctly and reducing the number of false positives. To reduce the number of false positives, I used a moving average (as previously mentioned). In doing so, I created a list of vehicle classes with a predefined length of 3. This means that my detection pipeline can only track up to 3 vehicle simultanuously. This could be extended easily, but is a current limitation.

Since the SVC was trained only on passenger cars in decent lighting conditions, it is very likely that this pipeline would fail in other lighting/environmental conditions (rain, night, snow, etc...) and would not identify other types of vehicles (motorcycles, pickups, trucks, etc...). To make the system more robust, more training data should be used which captures more vehicle types in different environmental conditions. To handle all these different scenarios, it is likely that several different classifiers would be needed (since, for example, a car at night looks much different than a car during the day).

This pipeline does not perform what would be a critical part of vehicle detection, evaluating the X and Y position of the vehicles. The bounding box gives a rough idea of where the vehicle is, but does not speficially identify the rear end. If a classifier were built that emphasized a very accurate identification of the bottom of the bounding box, then that could be assumed to be the intersection of the vehicle with the road. Lane lines could be used (like in the previous project) to calibrate a horizon, grid, and vanashing point which would reflect the 2D plane of a flat road. The aformentioned intersection of the bottom of the bounding box with the road would then provide the distance to the forward vehicle.