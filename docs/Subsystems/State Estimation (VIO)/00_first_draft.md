# First Draft
---

> [!NOTE]
> This a rough first draft with unstructured thoughts and experience during this project.
> Please ignore all the gramatical and spelling mistakes :)
---

## Visual Inertial Odometry (VIO)

Visual inertial odometry uses the camera footage and the IMU measurements to track the trajectory of the camera over time. 

There are multiple types of visual inertial odometry that can be done.
we don't strictly need visual inertial odometry, even visual odometry without the IMU can work. but that would require more than 1 camera to fix the scale ambiguity. Scale ambiguity comes with a monocular camera setup where we can track the movement. We can get the shape of the trajectory but we don't know the scale of the trajectory, like what does 1 unit in the odometry output mean in real world measurements. 
Along with this because we are using IMU data we can actually get a much faster update rate. Generally cameras are run at 30fps or lower or sometimes hight (with larger computational cost) but the IMU can give a very simple data efficient and computationally less expensive way to get a much higher pose estimate with hundereds of hertz of update rate without massively increasing the computational complexity.


there is Monocular VIO, and Stereo VIO, It is just as it sounds VIO using 1 camera and IMU is monocular VIO and using 2 cameras and 1 IMU is stereo VIO. 

Important hardware considerations for VIO
The camera should have a global shutter, especially if the trajectory can be rapid and quick (like in a drone) so that the rolling shutter distortion does not mess up the pose calculations
The camera and IMU data must be in sync, and in sync doesn't mean they both have to come at the same time (it won't be possible because both run at different frequency (camera at say 30fps) and Imu at more like 200Hz). In sync means the timestamp on both the camera and the IMU must be from the same clock. That is I can't put a timestamp on the data once my Compute unit gets the frames or the IMU data, I need timestamps of when the IMU data or camera frame was captured and the milliseconds matter here. Usually the IMU should be fine and quick, but the camera actually might have an added latency of a couple of milliseconds due to the internal processing overhead on the camera itself, that is why we can't time stamp it after getting the data, For this the cameras and IMU has external trigger pins that can help sync the clocks of both the devices. 
A large resolution camera is not necessary and might actually be very bad for VIO in compute constraint systems due to the shear computation that would be required to find and match features in a large resolution frame.
A native black and white camera is prefered because the VIO feature tracking does not depend on the colour of the features, it only calculates them based on the luminence and a native black and white camera would have a bigger light capture area per pixel for the same size sensor compared to a colour camera because in a colour camera each pixel have to be further divided into RGB spots hence reducing the light capture area, this would reduce the dynamic range of the camera and also increase the sensor noise. hence native BW cameras are better.


Calibration:
There are important paremeters that have to calibrated for the the camera and IMU.
Camera Calibration: (finding the intrinsic parameters of the camera):
There are multiple camera models to look out for, and each of them are best suited for certain types of cameras, The camera calibration is set to find the distortion parameters and the physical parameters like focal length. 
Since the camera I have is a stereo depth camera, I also calibrated the two cameras. This calibration find the extrinsic parameters of the two cameras that is the relative orientation and position of each of the cameras

IMU Calibration:
IMU calibration is not strictly necessary, we could just take the intrinsic parameters given by the manufacturer. but to get the most optimal performance we can do a manual IMU calibration. this involves collecting a long recording of the IMU at the particular frequency we'll be quiring the data while perferoming VIO, this recording should be at least a couple of hours long, 12 hours is a good amount of data. Then we run it through something called Allen Varience to find out the intrinsic parameters of the IMU. This gives a much more accurate picture of the IMU and this will be important to us because the surity in uncertainity will help us decide how well we should trust our IMU during the VIO state estimation.

Camera IMU calibration:
We also need to perform a camera IMU calibration. This is where we fidn out the extrinsic parameters of the IMU and the camera, that is the relative positioin and orientation of the IMU and Cameras, this is important. without this there would be slight offsets in the IMU and camera data that we can't account for because both of them are not on the same physical space. 



The steps followed generally depend on what type of VIO implementation that is done. The two methods are Filtering based and Optimization Based.
Filtering based methods were really simple to implement but they don't scale well with the number of features tracked that is their computational complexity increases by O(n^3). Filtering based methods are generally not as accurate as optimization based methods. In filtering based methods once we predict the next state we do not change the later as new information comes in. So whatever error we had before gets propegated to the new state. In optimization based techniques we optimize of a receding horizon of previous estimates, this means even if a prediction was wrong before it can be corrected as new information comes in. Hence optimization methods are more acurate but they consume a lot more computational power. There was a new filtering based apprached called MSCKF which reduced the filtering complexity to O(n). This was because the MSCKF state vector does not actually keep track of the feature positons in each frame. instead they use some sort of constraint(not sure here), which reduces the computational complexity.

Understanding a pure VO case is important for understanding VIO because VIO just uses VO estimates and fuses it with IMU estimates
This entire folowing section is based on the following two papers
Visual Odometry Part I - The First 30 Years and Fundametals
Visual Odometry Part II - Matching, Robustness, Optimization, and Application

For greater detail look into these papers. The following is just a quick summary of these 2 papers. 


Hence in VO the process to get the estimates we follow the following steps


-Feature Detection on the images

-Feature Matching (or Tracking)

-Motion Estimation (2D to 2D, 3D to 3D, 3D to 2D)

-Local Optimization

The Feature Detection itself can be done using many methods like ORB, SIFT, or direct tracking where the instead of detecting corners or somethign we just take features beased on their distinct luminance and this works really fast over a short span and does not really track a 3d feature from all angles especially when lighting changes. But it is extremely fast though

Feature Matching or Tracking - Feature Matching would be done by creating a descriptor for every single feature and then comparing the descriptors of features across frames and see if there is a close enough match, that is two descriptors from two different frames have a close enough to be considered similar (The close enough is a knob that we can dial in for our use case). The descriptors can be a floating point number or a vector, or it could be binary which is the fastest. 
Feature tracking finds the features in the first frame and then instead of seperately finding features on the next frame and then matching the features like feature matching. It would try and find the same features in the next frame that it tracked in the previous frame, and as we move along we would probably loose a lot of features hense a portion of new features will start to be detected if a lot of known features are lost. This works best if the successive frames have enough amount of overlap. one popular method is the KLT tracker 


`TODO` 

A breif about and RANSAC and outlier rejection
tell about the Mostion estimation, a brief of about 2D to 2D, 3D to 3D, 3D to 2D
A Brief obout local Optimzation.


Available Varients of VIO implementation:
ROVIO
VINS-Mono/VINS-Fusion
OpenVINS
TODO - give a brief about all of these implementations

I choose OpenVINS, due to its significantly less computational cost while running comparted to something like VINS-Mono,
At the same time I also wanted to use ROS2 as my software framework, and only OpenVINS had both ROS2 and ROS1 versians and ROS1 already reached endoflife in May of 2025 hence there is no more support for that framework.

What I did (i have to elobrate the following)
I calibrated the cameras and IMU using Kalibr tool
I did not yet do a proper IMU calibration alone and the imu parameters are taken from the manufacturer and so far the tracking has been really solid. 
There are much much more details about the implementation that I have. and I will write that in due time.


