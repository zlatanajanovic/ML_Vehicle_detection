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

[image1]: ./output_images/training_data_example.png "Example of car and noncar image from dataset"
[image2]: ./output_images/hog.png "HOG of a car image"
[image3]: ./output_images/training_data_normalized.png "Training data normalized"
[image4]: ./output_images/slide_window.png "Sliding window search (64x64)"
[image5]: ./output_images/subsampling.png "HOG subsampling"
[image6]: ./output_images/heatmap.png "Heat map"
[image7]: ./output_images/detection.png "Output image"
[video1]: ./project_marked.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

As the code includes also lane detection from previous project, the code for this step is contained in the 20th code cell of the IPython notebook (n the function `get_hog_features()`).  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`). 

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image2]


#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and this one showed to be the most robust, with the least false positives.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using all 8000+ images for each car and noncar images. For HOG features `YCrCb` color space with all channels and `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`. For color features `spatial_size = (32, 32)` was used with `hist_bins = 32` for color histogram. All of the features are combined and scaled to be used for training classifier. On the image bellow you can see how does it look before and after scaling.

![alt text][image3]

Finally, trained classifier had `Feature vector length: 8460`, `26.36 Seconds to train SVC...` and `Test Accuracy of SVC =  0.9901`

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I first tested classifier using sliding window. I tried different sizes to see how it detects. Here is an example with fixed `windows size 64`. It can be seen that the white vehicle is not detected because it is bigger. Therefore in final search I used different sizes and added them together.

![alt text][image4]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

To speed up search I used HOG features subsampling. In this way calculating HOG features, which is very slow, is done only once and than windows of interest are sampled from here. Also I searched only lower part of the image, because the cars could be only on this part of the image. Here are some example images for HOG subsampling using `scale 1.2`:

![alt text][image5]

Ultimately I searched on 4 scales `(1, 1.5, 2, 2.5 which corresponds to window size 64, 96, 128, 150 pixels)` using `YCrCb` 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  


---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a one image, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

Here is the frame and its corresponding heatmap:

![alt text][image6]

Ultimately I used `10 frames` and added them to same heatmap and used threshold to avoid false positives. I also integrated lane detection from previous project. Here the resulting bounding boxes are drawn onto the last frame in the series:

![alt text][image7]


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Most of the problems I faces were caused by bugs in a code an not by missing some concept. Additionally because of the many parameter which influence overlapping behaviours it is not easy to adjust it to work nice. For example increasing threshold for avoiding false positives also affects vehicle detection. Also, computation time is not very short and tastings consume a lot of time. To use it practically the processing time should be decreased at least for 2 orders of magnitude. 
Averaging amongst frames could be improved with object tracking (for example Kalman filter)an this could also cover some vehicle dynamics. For this if would be useful to go to bird-view perspective after vehicle detection and calculate vehicle position and relative speed and implement some tracking algorithm. Current approach is not adaptable, it would be useful to implement some adaptation on parameters to the environmental conditions.


