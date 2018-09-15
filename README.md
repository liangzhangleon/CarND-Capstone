# System integration project

This is the project repo for the final project of the Udacity Self-Driving Car Nanodegree: Programming a Real Self-Driving Car. For more information about the project, see the project introduction [here](https://classroom.udacity.com/nanodegrees/nd013/parts/6047fe34-d93c-4f50-8336-b70ef10cb4b2/modules/e1a23b06-329a-4684-a717-ad476f0d8dff/lessons/462c933d-9f24-42d3-8bdc-a08a5fc866e4/concepts/5ab4b122-83e6-436d-850f-9f4d26627fd9).

Please use **one** of the two installation options, either native **or** docker installation.

### Individual submission
Name | Udacity account
--- | ---
Liang Zhang | liangzhangleon@outlook.com

## ROS node introduction
The following is the overall system architecture.

![sys_architec](imgs/sys_architec.png)

I will only introduce three nodes: waypoint_updater, dbw_note and tl_detector.
### waypoint_updater
Below input and output topics of this node is described.
![sys_architec](imgs/waypoint_updater.png)

In waypoint_updater.py, I completed this node in two steps.
#### Step one: without traffic get_light_state
Without traffic lights, the main functionality I implemented is calculating the next 50 waypoints ahead of the ego car based on the base waypoints. The important function is get_closest_waypoint_idx().

#### Step two: with traffic light
With traffic lights, once a traffic light is detected, a stopline_wp_idx is given. With this stopline_wp_idx, decelerate_waypoints(...) function is used to get a group deceleration waypoints with target speed gradually approaches zero.

### dbw_node
Below input and output topics of this node is described.
![sys_architec](imgs/dbw_node.png)

#### dbw_node.py
In this file, the three callback functions are simple. One just needs to load and assign the messages.
In the loop() functions, one call the controller.control(...) to calculate the brake, steering and throttle needed, then publish them.

#### twist_controller.py
I use YawController for steering and PID controller for throttle. The brake is computed via the formula
```
            brake = abs(decel) * self.vehicle_mass * self.wheel_radius
```
where decel is deceleration speed.

### tl_detector
Below input and output topics of this node is described.
![sys_architec](imgs/tl_detector.png)
#### tl_detector.py
The main functionality of this node starts from image_cb(...), in which process_traffic_lights() function will find the closest traffic light and then determines the color of the traffic light.

#### tl_classifier.py
The tl_classifier will take the camera image as input, and load the trained Tensorflow model (simulation or site). The process to train the model will be explained in next section. The classifier will first pre-process the image, predict the color state, then output the state to tl_detector.

#### Traffic light detection/classification model
I followed instructions given in
[Tensorflow's Object Detection API](https://github.com/coldKnight/TrafficLight_Detection-TensorFlowAPI).

I used a pre-trained tensorflow model [here](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md). This model is called *ssd_mobilenet_v1_coco* in the COCO-trained models.

The model is re-trained with the image dataset was downloaded from the [link](https://drive.google.com/file/d/0B-Eiyn-CUQtxdUZWMkFfQzdObUE/view?usp=sharing) given in this [repository](https://github.com/coldKnight/TrafficLight_Detection-TensorFlowAPI).



## Note and outlook
### Note
* waypoint_updater does not give a warning or an error that it can not update waypoints in time. When I use a slow computer to run the simulation, the controller will not control the car properly when waypoints were not updated.

### Outlook
* The traffic light detector/classifier occasionally has false positive error for yellow object, which should be improved.  



## Project instructions

### Native Installation

* Be sure that your workstation is running Ubuntu 16.04 Xenial Xerus or Ubuntu 14.04 Trusty Tahir. [Ubuntu downloads can be found here](https://www.ubuntu.com/download/desktop).
* If using a Virtual Machine to install Ubuntu, use the following configuration as minimum:
  * 2 CPU
  * 2 GB system memory
  * 25 GB of free hard drive space

  The Udacity provided virtual machine has ROS and Dataspeed DBW already installed, so you can skip the next two steps if you are using this.

* Follow these instructions to install ROS
  * [ROS Kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu) if you have Ubuntu 16.04.
  * [ROS Indigo](http://wiki.ros.org/indigo/Installation/Ubuntu) if you have Ubuntu 14.04.
* [Dataspeed DBW](https://bitbucket.org/DataspeedInc/dbw_mkz_ros)
  * Use this option to install the SDK on a workstation that already has ROS installed: [One Line SDK Install (binary)](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/src/81e63fcc335d7b64139d7482017d6a97b405e250/ROS_SETUP.md?fileviewer=file-view-default)
* Download the [Udacity Simulator](https://github.com/udacity/CarND-Capstone/releases).

### Docker Installation
[Install Docker](https://docs.docker.com/engine/installation/)

Build the docker container
```bash
docker build . -t capstone
```

Run the docker file
```bash
docker run -p 4567:4567 -v $PWD:/capstone -v /tmp/log:/root/.ros/ --rm -it capstone
```

### Port Forwarding
To set up port forwarding, please refer to the [instructions from term 2](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/16cf4a78-4fc7-49e1-8621-3450ca938b77)

### Usage

1. Clone the project repository
```bash
git clone https://github.com/udacity/CarND-Capstone.git
```

2. Install python dependencies
```bash
cd CarND-Capstone
pip install -r requirements.txt
```
3. Make and run styx
```bash
cd ros
catkin_make
source devel/setup.sh
roslaunch launch/styx.launch
```
4. Run the simulator

### Real world testing
1. Download [training bag](https://s3-us-west-1.amazonaws.com/udacity-selfdrivingcar/traffic_light_bag_file.zip) that was recorded on the Udacity self-driving car.
2. Unzip the file
```bash
unzip traffic_light_bag_file.zip
```
3. Play the bag file
```bash
rosbag play -l traffic_light_bag_file/traffic_light_training.bag
```
4. Launch your project in site mode
```bash
cd CarND-Capstone/ros
roslaunch launch/site.launch
```
5. Confirm that traffic light detection works on real life images
