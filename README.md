# Robotic Catheter Ablation: An Evaluation and Prototyping Platform
Welcome to the setup guide for "Robotic Catheter Ablation: An Evaluation and Prototyping Platform." In this guide, we will walk you through the steps of setting up evaluation paltform for catheter ablation. Following this guide will help you calibrate and register your camera system effectively and setting up the physical platform. Let's start by gathering all the necessary requirements.


## 1. Requirements
### Software requirements
- [ROS Noetic](http://wiki.ros.org/noetic/Installation/Ubuntu)
- [usb_cam](http://wiki.ros.org/usb_cam)
- [apriltag_ros](http://wiki.ros.org/apriltag_ros)
- Matlab's [Stereo Camera Calibrator](https://www.mathworks.com/help/vision/ug/using-the-stereo-camera-calibrator-app.html)

### Further requirments
- 2x endoscopic camers
- 1x 3D-printed [STL-Files](./stl)
- 1x Printed [Apriltag](./misc/apriltag.pdf)
- 1x Printed [Checkerboard](./misc/checkerboard.pdf)
- 1x Printed [Validation Marker](./misc/tracking_validation.pdf)
- 8x M3 bolts
- 6x M3 nuts

## 2. Stereo Camera Setup
### Camera placement
The two endoscopic cameras are placed in the [main part](./stl/main_part.STL) of the physical setup as follows:
![Apriltag 1](./images/main_part_stereo.JPG) 

_Figure 1: An illustration of the stereo camera setup showcasing the correct placement of endoscopic cameras._

Utilize M3 bolts to secure the cameras firmly in the model to prevent any movements during operation.

### Stream camera videos
To stream the videos from the two cameras, we utilize the [usb_cam](http://wiki.ros.org/usb_cam) package in ROS Noetic. The following launch file starts the two cameras and publishes the video streams to `stereo_cam_right/image_raw` and `stereo_cam_left/image_raw`:
```xml
<launch>
  <!-- Arguments -->
  <arg name="frame_rate" default="25" /> <!-- Frame rate of the video streams -->
  <arg name="pixel_format" default="yuyv" /> <!-- Pixel format of the video streams -->

  <!-- Launch right camera -->
  <node name="stereo_cam_right_node" pkg="usb_cam" type="usb_cam_node">
    <param name="camera_name" value="stereo_cam_right" />
    <param name="camera_frame_id" value="/stereo_cam_right" />
    <param name="video_device" value="/dev/<PATH_RIGHT_CAMERA>"/> <!-- Replace with the actual path to your right camera -->
    <param name="pixel_format" value="$(arg pixel_format)" />
    <param name="framerate" value="$(arg frame_rate)" /> 
  </node>

  <!-- Launch left camera -->
  <node name="stereo_cam_left_node" pkg="usb_cam" type="usb_cam_node">
    <param name="camera_name" value="stereo_cam_left" />
    <param name="camera_frame_id" value="/stereo_cam_left" />
    <param name="video_device" value="/dev/<PATH_LEFT_CAMERA>"/> <!-- Replace with the actual path to your left camera -->
    <param name="pixel_format" value="$(arg pixel_format)" />
    <param name="framerate" value="$(arg frame_rate)" /> 
  </node>
</launch>

```

Please ensure to replace <PATH_RIGHT_CAMERA> and <PATH_LEFT_CAMERA> with the appropriate paths to your right and left cameras, respectively. 

### Stereo Camera Calibration
Calibrating the stereo camera involves determining the intrinsic and extrinsic parameters of the individual cameras. To accomplish this, we utilize the Matlab's [Stereo Camera Calibrator](https://www.mathworks.com/help/vision/ug/using-the-stereo-camera-calibrator-app.html). The [checkerboard](./misc/checkerboard.pdf) is used to calibrate the cameras by placing it at different orientations and distances with respect to the cameras. Capture multiple pairs of synchronized images of the checkerboard using both the left and right cameras. Make sure to cover various angles and distances to get a comprehensive set of data for calibration:

![Checkerboard](./images/checkerboard.png)

_Figure 2: An illustration of the stereo camera calibration using Matlab's Stereo Camera Calibrator._


### April Tag Registration
To obtain the transformation from the stereo camera setup to the model, we utilize AprilTags, a type of fiducial marker system. Follow the steps below to set up the registration:

#### **Preparation of the AprilTag**: 
- Print the [Apriltag marker](./misc/apriltag.pdf).
- Attach it to the registration unit as illustrated in Figure 3.
   
![Apriltag](./images/apriltag.JPG)
   
_Figure 3: AprilTag attached to the registration unit._

#### **Attaching the Registration Unit**: 
- Attach the registration unit to the main part of the model to allow for the extraction of transformation data.

#### **Launching the AprilTag Detection**:
- Utilize the [apriltag_ros package](http://wiki.ros.org/apriltag_ros) for detecting the AprilTags in both the left and right camera streams. Below is the launch file that initiates the AprilTag detection nodes:

 ```xml
<launch>
  <!-- Arguments -->
  <arg name="camera_name_left" default="/stereo_cam_left" />
  <arg name="camera_name_right" default="/stereo_cam_right" />
  <arg name="apriltag_definition_file" default="<PATH_TO_DEFINITION>" />
  <arg name="apriltag_settings_file" default="<PATH_TO_SETTINGS>" />

  <!-- Launch AprilTag detection for the right camera -->
  <group ns="$(arg camera_name_right)"> 
    <node pkg="image_proc" type="image_proc" name="image_proc" />
    <node pkg="apriltag_ros" type="apriltag_ros_continuous_node" name="apriltag_right_cam" clear_params="true" output="screen">
      <rosparam command="load" file="$(arg apriltag_settings_file)" />
      <rosparam command="load" file="$(arg apriltag_definition_file)" />
      <param name="camera_frame" type="str" value="$(arg camera_name_right)" />
    </node>
  </group>

  <!-- Launch AprilTag detection for the left camera -->
  <group ns="$(arg camera_name_left)"> 
    <node pkg="image_proc" type="image_proc" name="image_proc" />
    <node pkg="apriltag_ros" type="apriltag_ros_continuous_node" name="apriltag_left_cam" clear_params="true" output="screen">
      <rosparam command="load" file="$(arg apriltag_settings_file)" />
      <rosparam command="load" file="$(arg apriltag_definition_file)" />
      <param name="camera_frame" type="str" value="$(arg camera_name_left)" />
    </node>
  </group>
</launch>
```

### Accuracy Evaluation

To evaluate the accuracy of the stereo camera setup, we triangulate two color markers. We estimate the 3D coordinates of the markers and compare them to their known positions to determine the system's accuracy. The following figure illustrates how the color marker pattern is placed in the registration unit:

![Accuracy Evaluation](./images/tracking_validation.JPG)

_Figure 4: Evaluation color markers attached to the registration unit._

The known positions of the markers in the AprilTag frame are:
  - **Green marker**: [0,0,0]
  - **Red marker**: [0.005,0,0]

## 3. GUI

In this section, we cover the GUI setup, which includes loading the heart model and setting up the static transformations. It is essential to correctly set up the GUI to visualize and interact with the heart model appropriately.


### Load heart model

To load the heart model, define the color and the necessary visual parameters using the following XML setup:

```xml
<robot name="visual">
  <!-- Define color -->
  <material name="red">
    <color rgba="1 0.3 0.3 1"/>
  </material>

  <link name="pulmonary_vein_main_fixture">

    <!-- Main fixture -->
    <visual> 
      <geometry>
        <mesh filename="<PATH_TO_MAIN_PART_STL>" />
      </geometry>
      <material name="red"/>
    </visual>

    <!-- Ablation section  -->
    <visual>
      <geometry>
        <mesh filename="<PATH_TO_ABLATION_SECTION_STL>" />
      </geometry>
      <material name="red"/>
    </visual>

  </link>

</robot>
```

Next, establish the necessary launch parameters and transformations using the snippet below:

```xml
<launch>
  <!-- Load model  -->
  <arg name="model" default="<PATH_TO_URDF>"/>
  <arg name="gui" default="true"/>
  <param name="pulmonary_vein" command="$(find xacro)/xacro $(arg model)" />

  <!-- Transformation from the model to the world frame  -->
  <node pkg="tf" type="static_transform_publisher" name="pulmonary_vein_broadcaster" args="0 0 0 0 0 0 1 mns model 60" />
</launch>
```

### Static transformations

The following XML script helps in establishing the necessary static transformations from the model to the AprilTag to the camera setup:

```xml
<launch>
  <!-- These nodes publish the static transformations form the AprilTag -> ApriltTag Mount -> PV model. They are know from the geometry of the calibration unit and the AprilTag -->
  <node pkg="tf" type="static_transform_publisher" name="model_to_apriltag_center_tf" args="0.0908 0.0844 0.08045 0 0 0 1 model tag_0_center 60" />
  <node pkg="tf" type="static_transform_publisher" name="apriltag_center_to_apriltag_tf" args="0 0.007 0 0.5 -0.5 -0.5 0.5 tag_0_center tag_0 60" />

  <!-- Broadcast the transformation from the AprilTag to the camera frame-->
  <node pkg="tf" type="static_transform_publisher" name="apriltag_to_camera_tf" args="<PLACEHOLDER> tag_0 stereo_cam_right 60" />
</launch>
```

## 4. Physical Setup

In this section, we delineate the various components of the physical setup, including the placement of the catheter and the complete arrangement facilitating catheter tracking.

### Catheter Placement

The catheter is positioned to enable accurate tracking. Refer to the image below for a visualization of its placement:

![Catheter](./images/catheter.JPG)

_Figure 5: The catheter as positioned in the physical setup._

### Full Setup

Next, we present the complete setup, which integrates all the individual components, preparing it for the catheter tracking process as demonstrated in the subsequent section:

![Full Setup](./images/full_setup.JPG)

_Figure 6: The complete setup, showcasing all the components properly assembled._

### Catheter Tracking Demonstration

To further elucidate the functionality of the setup, we have recorded a GIF demonstrating the real-time catheter tracking. This visualization aids in comprehending the operational flow of the setup:

![Tracking GIF](./images/tracking.gif) 

_Figure 7: A GIF demonstrating the catheter tracking in real-time, highlighting the setup’s operational efficacy._

By following the detailed setup guide and utilizing the documented physical setup, one can effectively replicate the system for catheter tracking using stereo camera and AprilTag registration.

## 5. Reference

For more detailed instructions, the reader is refered to the following publication:

```
@inproceedings{Heemeyer2023,
  doi = {10.1109/ismr57123.2023.10130271},
  url = {https://doi.org/10.1109/ismr57123.2023.10130271},
  year = {2023},
  month = apr,
  publisher = {{IEEE}},
  author = {Florian Heemeyer and Christophe Chautems and Quentin Boehler and Jos{\'{e}} L. Merino and Bradley J. Nelson},
  title = {An Evaluation Platform for Catheter Ablation Navigation},
  booktitle = {2023 International Symposium on Medical Robotics ({ISMR})}
}
```
