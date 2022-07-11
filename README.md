# SFND 3D Object Tracking
By completing all the lessons, a solid understanding of keypoint detectors, descriptors, and methods to match them between successive images have been implemented. Also, you know how to detect objects in an image using the YOLO deep-learning framework. 
And finally, understand how to associate regions in a camera image with Lidar points in 3D space. This project will integrate all those techniques to tracking objects in a 3D space.

![result](https://user-images.githubusercontent.com/31724244/178157698-80c9d515-11d5-4255-b944-eacbcc6bae7d.gif)


In this final project, you will implement the missing parts in the schematic. To do this, you will complete four major tasks: 
1. First, you will develop a way to match 3D objects over time by using keypoint correspondences. 
2. Second, you will compute the TTC based on Lidar measurements. 
3. You will then proceed to do the same using the camera, which requires to first associate keypoint matches to regions of interest and then to compute the TTC based on those matches. 
4. And lastly, you will conduct various tests with the framework. Your goal is to identify the most suitable detector/descriptor combination for TTC estimation and also to search for problems that can lead to faulty measurements by the camera or Lidar sensor. In the last course of this Nanodegree, you will learn about the Kalman filter, which is a great way to combine the two independent TTC measurements into an improved version which is much more reliable than a single sensor alone can be. But before we think about such things, let us focus on your final project in the camera course. 


## Dependencies for Running Locally
* cmake >= 2.8
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1 (Linux, Mac), 3.81 (Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* Git LFS
  * Weight files are handled using [LFS](https://git-lfs.github.com/)
* OpenCV >= 4.1
  * This must be compiled from source using the `-D OPENCV_ENABLE_NONFREE=ON` cmake flag for testing the SIFT and SURF detectors.
  * The OpenCV 4.1.0 source code can be found [here](https://github.com/opencv/opencv/tree/4.1.0)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools](https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory in the top level project directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./3D_object_tracking`.


### FP.0 Match 2D Objects
In the matching2D_Student.cpp, the mid-term project code to detect keypoints, extract descriptors and match descriptors is applied. Various type of detectors and descriptors are implemented. 

### FP.1 Match 3D Objects
In the camFusion_Student.cpp, the matchBoundingBoxes method is implemented. The input include the previous and current datat frames and matches keypoint between these two frame. Only bounding boxes containing matches keypoints within the ROI filter are considered. Only one of the best result shown associated bounding boxes with the highest number of occurrences is extracted. 

### FP.2 Compute Lidar-based TTC
In the camFusion_Student.cpp, computeTTCLidar() the time-to-collision is implemented based on lidar points within the region of interest of each matched boundingbox. To filter out the outliers of lidar points that are happens between ego and target vehicle as noise, instead of using the average distance of lidar points, the same concept of using median x point as used in computeTTCCamera() within the ROI is used to calculate teh TTC for more robust estimation.  

### FP.3 Associate Keypoint Correspondences with Bounding Boxes
In the camFusion_Student.cpp, clusterKptMatchesWithROI(). The keypoints in previous and current frame are matched and checked if they are within the ROI. To have a robust TTC estimation, the average eucliean mean distance is calcualte and used to filer out the outliers of the matches. 

### FP.4 Compute Camera-based TTC
In the camFusion_Student.cpp, the time-to-collision in seconds is implemented using the relationship established between keypoint correspondences between previous and current frames.

## Write up
After finishing the pipeline in the FinalProject_Camera.cpp, the TTC estimations based on both Lidar points and Camera images is generated. 

### FP.5 Performance Evaluation 1
An example where the Lidar TTC estimate is inaccurate is when lidar  is not always correct because of some outliers and some unstable points from preceding vehicle's  but much closer than that. e.g. the ttc from lidar jumps from 12.8s to 17.6s in betwwen to close frames
![result_0002](https://user-images.githubusercontent.com/31724244/178157947-b999ed0e-a943-44eb-823b-39cdecaacbb4.png)
![result_0003](https://user-images.githubusercontent.com/31724244/178157960-c0400ff7-673d-4074-95c4-8a39b9268af2.png)
even though the minimum distance decreased from 7.79m to 7.68m. These points can be filtered out simply by increasing the shrink factor when clustering the lidar with ROI. e.g. change the shrink factor from 0.1 to 0,2 or higher

FIX**: after implementing the median value instead of mean, the Lidar TTC results is relatively robust and reasonable since the impact of the outliers is significantly reduced. 
However there are a few frames of result that are shown a higher std, this is mainly due to be simplified 1st order CV model that's used to calculate TTC since Lidar doesn't provide doppler measurement directly. 
![result_0002](https://user-images.githubusercontent.com/31724244/178166670-fc7b960a-075b-4179-bc6f-59093776663a.png)
![result_0003](https://user-images.githubusercontent.com/31724244/178166673-b05d91bb-f733-437c-9f0d-065ed16c382a.png)

### FP.6 Performance Evaluation 2
As learnt from the mid-term project, various test of different combinations of detector/descriptors were tested in terms of performance vs accuracy and speed.
e.g. for the 1st and 2nd frame using Lidar TTC(s) as ground truth: 12.23. The related best three combination shown most accurate (using Lidar as GT) are AKAEZ+BRISK; FAST+BRISK, BRISK+ORB

| Camera TTC(s)           |             |         |             |         |         |         |
| ----------------------- | ----------- | :-----: | :---------: | :-----: | :-----: | :-----: |
| **Detector/Descriptor** | BRISK       |  BRIEF  |     ORB     |  FREAK  |  AKAZE  |  SIFT   |
| SHI-TOMASI              | 10.4544     | 11.5702 |   10.9314   | 11.8606 |    X    | 12.4518 |
| HARRIS                  | non         | 8.81335 |   10.9082   | 8.75397 |    X    | 8.81335 |
| FAST                    | **12.555**  | 10.6507 |   11.6137   | 11.7545 |    X    |  11.99  |
| BRISK                   | 12.8364     | 10.5599 | **12.3924** | 10.9016 |    X    | 15.5841 |
| ORB                     | 12.1791     | 16.4808 |   19.7314   | 12.2074 |    X    | 9.83629 |
| AKAZE                   | **12.2628** | 12.4103 |   11.4295   | 11.6351 | 11.8071 | 12.273  |
| SIFT                    | 11.8926     | 11.8654 |      X      | 10.5213 |    X    | 10.2713 |


Similar as Lidar, Camera TTC is not alway accurate or robust. This is normally due to poor keypoint matching, or the keypoint descriptor is not good enough to discriminate keypoints which are clustered together.  e.g. the ttc from camera jumps from 16s to 26s in betwwen to close frames when using poor combination of detector&descriptor such as Harris + ORB. 
![result_0007](https://user-images.githubusercontent.com/31724244/178158452-72b035be-0d0f-413c-b864-026b5c4e8e92.png)
![result_0008](https://user-images.githubusercontent.com/31724244/178158456-e22151a8-9b54-4de6-a648-6ed654ae58d5.png)

Finally, the average of the TTC calculated along the whole set of frames is shown below, the HARRIS is not considered in the comparision due to unrealibale performance. 
|Detector| Descriptor | Difference in TTC | Lidar TTC Average | Camera TTC Average |
|--------|------------|-------------------|-------------------|-------------------|
|SHITOMASI |BRISK|3.90278|11.7444|12.2111|
|FAST|BRIEF|3.58979|11.7444|**12.1051**|
|FAST|ORB|5.29804|11.7444|12.5039|
|AKAZE|AKAZE|3.69089|11.7444|12.2384|
|SIFT|SIFT|3.28828|11.7444|**11.7344**|
|FAST|FREAK|6.74713|11.7444|**11.8533**|
As shown in the result, the top 3 detector/descriptor combinations, that achieve most close result to Lidar TTC in average are: SIFT+SIFT; FAST+FREAK; FAST+BRIEF. 

