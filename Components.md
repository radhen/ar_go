# Components of the autonomous car

    1. Chasis - 
    2. Camera - oCam
    3. IR sensor - SHARP
    4. ODroid - running Ubuntu MATE 16.04
    5. Wifi module - plug and play
    6. IMU - 
    7. Servo controller - 
    8. ESC - 
    9. Buck converter



## Chasis


## Camera - Yang Li
oCam-iMGN-U by www.withrobot.com

- 1MP USB 3.0 Mono Camera
- Global shutter
- High speed up to 160 frames-per-second at the 320 x 240 resolution
- UVC compliance

### How to run oCam-viewer
https://github.com/withrobot/oCam/tree/master/Software/oCam_viewer_Linux

build it and then 
```bash
$ cd OCAM_VIEWER_BUILD_DIRECTORY
$ ./oCam-viewer
```
### How to get image?

USB --> Kernel (through xhci and V4L2) --> User Space (ROS)

```bash
ls /dev/video0
```
make sure the camera is connected correctly

```bash
sudo apt-get update
sudo apt-get install guvcview
```
install camera view application guvcview. Run it and you should be able to see the image catched by your camera.

Get started with the following launch file. If you need assistance with your webcam's supported resolutions and frame rates, try:
```bash
v4l2-ctl --list-formats-ext
```
Installing "V4l2-ctl" on Ubuntu

Here's how to install "V4l2-ctl" on Ubuntu Linux operating system:
```
sudo apt-get install v4l-utils
```

### Before installing libuvc_ros, we should install libuvc first

Use the updated version mentioned in the libuvc_camera folder of https://github.com/AdvancedRoboticsCUBoulder/libuvc_ros

As talked in http://answers.ros.org/question/204840/libuvc_ros-not-building
libuvc is a library that supports enumeration, control and streaming for USB Video Class (UVC) devices, such as consumer webcams.

```bash
$ git clone https://github.com/ktossell/libuvc.git
$ cd libuvc
$ mkdir build
$ cd build
$ cmake ..
$ make && sudo make install
```

### Install libuvc_ros

Download the souce code from https://github.com/ktossell/libuvc_ros to your ROS workspace and build it.

The documentation is in http://wiki.ros.org/libuvc_camera

```bash
lsusb
Bus 003 Device 002: ID 04b4:00f8 Cypress Semiconductor Corp. 
```
To get the vendor and product IDs
```bash
lsusb -d 04b4:00f8 -v

Bus 003 Device 002: ID 04b4:00f8 Cypress Semiconductor Corp. 
Couldn't open device, some information will be missing
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass          239 Miscellaneous Device
  bDeviceSubClass         2 ?
  bDeviceProtocol         1 Interface Association
  bMaxPacketSize0        64
  idVendor           0x04b4 Cypress Semiconductor Corp.
  idProduct          0x00f8 
  bcdDevice           16.11
...
``` 

```bash
roslaunch libuvc_camera libuvc.launch 
```

```xml
<launch>
  <arg name="debug" default="false"/>
  <arg name="launch_prefix" if="$(arg debug)" default="gdb -e run --args"/>
  <arg name="launch_prefix" unless="$(arg debug)" default=""/>
  <!-- output="screen" -->
  <group ns="camera">
    <node pkg="libuvc_camera" type="camera_node" name="ocam" output="screen" launch-prefix="$(arg launch_prefix)">
      <!-- Parameters used to find the camera -->
      <param name="vendor" value="0x04b4"/>
      <param name="product" value="0x00f8"/>
      <param name="serial" value=""/>
      <!-- If the above parameters aren't unique, choose the first match: -->
      <param name="index" value="0"/>

      <!-- Image size and type -->
      <param name="width" value="640"/>
      <param name="height" value="480"/>
      <!-- choose whichever uncompressed format the camera supports: -->
      <param name="video_mode" value="gray8"/> <!-- or yuyv/nv12/jpeg -->
      <param name="frame_rate" value="80"/>

      <param name="timestamp_method" value="start"/> <!-- start of frame -->
      <!-- <param name="camera_info_url" value="file:///tmp/cam.yaml"/> -->

<!--       <param name="auto_exposure" value="3"/> use aperture_priority auto exposure
      <param name="auto_white_balance" value="false"/> -->
    </node>
  </group>
</launch>
```

**Topics got from running the libuvc_camera**
```bash
yang@colab:~$ rostopic list
/camera/camera_info
/camera/image_raw
/camera/image_raw/compressed
/camera/image_raw/compressed/parameter_descriptions
/camera/image_raw/compressed/parameter_updates
/camera/image_raw/compressedDepth
/camera/image_raw/compressedDepth/parameter_descriptions
/camera/image_raw/compressedDepth/parameter_updates
/camera/image_raw/theora
/camera/image_raw/theora/parameter_descriptions
/camera/image_raw/theora/parameter_updates
/camera/ocam/parameter_descriptions
/camera/ocam/parameter_updates
/rosout
/rosout_agg
```

**Results from running the launch file**
```bash
yang@colab:~$ roslaunch libuvc_camera libuvc.launch 
... logging to /home/yang/.ros/log/d49538ee-10d9-11e7-9400-d43d7eb70ebb/roslaunch-colab-15588.log
Checking log directory for disk usage. This may take awhile.
Press Ctrl-C to interrupt
Done checking log file disk usage. Usage is <1GB.

started roslaunch server http://localhost:44780/

SUMMARY
========

PARAMETERS
 * /camera/ocam/frame_rate: 80
 * /camera/ocam/height: 480
 * /camera/ocam/index: 0
 * /camera/ocam/product: 0x00f8
 * /camera/ocam/serial: 
 * /camera/ocam/timestamp_method: start
 * /camera/ocam/vendor: 0x04b4
 * /camera/ocam/video_mode: gray8
 * /camera/ocam/width: 640
 * /rosdistro: kinetic
 * /rosversion: 1.12.6


NODES
  /camera/
    ocam (libuvc_camera/camera_node)

auto-starting new master
process[master]: started with pid [15599]
ROS_MASTER_URI=http://localhost:11311

setting /run_id to d49538ee-10d9-11e7-9400-d43d7eb70ebb
process[rosout-1]: started with pid [15612]
started core service [/rosout]
log4cxx: Could not read configuration file [~/rosconsole.config].
process[camera/ocam-2]: started with pid [15629]
log4cxx: Could not read configuration file [~/rosconsole.config].
 INFO ros.libuvc_camera: Opening camera with vendor=0x4b4, product=0xf8, serial="", index=0
unsupported descriptor subtype: 13
 WARN ros.libuvc_camera: Unable to set auto_exposure to 8
 WARN ros.libuvc_camera: Unable to set auto_exposure_priority to 0
 WARN ros.libuvc_camera: Unable to set auto_focus to 1
 WARN ros.libuvc_camera: Unable to set focus_absolute to 0
 WARN ros.libuvc_camera: Unable to set gain to 0
 WARN ros.libuvc_camera: Unable to set iris_absolute to 0
 WARN ros.libuvc_camera: Unable to set pantilt to 0, 0
```

### Run Steve's simple_opencv node

https://github.com/AdvancedRoboticsCUBoulder/simple_opencv

Remeber to modify CMakelists.txt about OpenCV3

```
#If you have a ROS-local install of OpenCV3:
set(TMP_PREFIX_PATH ${CMAKE_PREFIX_PATH})
set(CMAKE_PREFIX_PATH "/opt/ros/indigo/share/OpenCV-3.1.0-dev")
find_package(OpenCV 3.0 REQUIRED)
```

There is also tutorial about publishing and subscribing images on http://wiki.ros.org/image_transport/Tutorials


**uvcdynctrl** libwebcam command line tool


### Try to use the Camera class of oCam


## IR sensors - Jay and Yang Li
SHARP 68 2Y0A710 F

Two ground and 5V are needed.


## IMU - Jay
```bash
TODO
```