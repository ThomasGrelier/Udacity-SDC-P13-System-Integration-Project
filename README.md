# UDACITY - System Integration Project
---

Programming a Real Self-Driving Car for the UDACITY Nanodegree


## The Happy Crashers Team:

|     Image              |     Name      |  LinkedIn    |     email   |
|------------------------|---------------|----------------|---------------|
| <img src="./assets/c_gordillo.jpg" alt="Camilo Gordillo" width="150" height="150"> | Camilo Gordillo | [Camilo](https://de.linkedin.com/in/camilogordillo/en) | <camigord@gmail.com> |
| <img src="./assets/s_salati.jpg" alt="Stefano Salati" width="150" height="150"> | Stefano Salati | [Stefano](https://www.linkedin.com/in/stefanosalati/) | <stef.salati@gmail.com> |
| <img src="./assets/s_rademacher.jpg" alt="Stefan Rademacher" width="150" height="150"> | Stefan Rademacher | [Stefan](https://www.linkedin.com/in/stefan-rademacher/) | <rademacher@outlook.com> |
| <img src="./assets/t_grelier.jpg" alt="Thomas GRELIER" width="150" height="150"> | Thomas GRELIER | [Thomas](https://www.linkedin.com/in/thomas-grelier/) | <masto.grelier@gmail.com> |

## Project description

The objective of this project was to write ROS nodes to implement core functionality of the autonomous vehicle system. This included traffic light detection, waypoint planner and control.

### Control strategy
The car behaviour is regulated by a FSM with two states: normal driving and braking.

The car drives normally when no traffic light is detected within a range of 100m or when the detected traffic light is green. This is obtained by setting the speed of all waypoints ahead to their default values.

As soon as a red traffic light is detected, the system computes the minimum braking distance to check if a braking is possible. If yes, a braking deceleration - dependent on the current speed and distance of the traffic light - is applied and the speed of all waypoints between the car and the traffic light is set so it gently decrease to zero at the light. All speeds of waypoints ahead of the traffic light are set to zero.

<img src="./assets/a_brake.gif" alt="a_{brake} = \frac{v_{car}^2}{2*d_{light}}">

<img src="./assets/v_i.gif" alt="v_{i} = \left\{\begin{matrix}  \sqrt{2*a_{brake} * d_{i}} & \textrm{for waypoints before traffic light} \\ 0 & \textrm{for waypoints beyond traffic light} \end{matrix}\right.">

It's worth noting this scenario: a red light is detected and the car starts braking gently, if the light turns green while the car is braking, the car switches back to normal driving and accelerates to normal speed. This replicates the usual human behavior of not waiting the last moment to brake but to let go as soon as a red light is seen.

Throttle and brake pedal are then controlled by a PID, tuned with:

| Parameter | Value  |
|-----------|--------|
| VEL_PID_P | 0.8    |
| VEL_PID_I | 0.0001 |
| VEL_PID_D | 0.01   |

### Traffic Light Classification

We assume that there is only one traffic light color in Carlas field of view at the same time. Thus the detection with the highest score gets selected to identify the traffic light color. If there is no detection with a score higher than 50% probability, the classifier function returns "UNKNOWN".
The detection itself is done using a Convolutional Neural Network. We have used the TensorFlow Object Detection API and detection models, that are pre-trained on the COCO dataset. To be exact we have used two different models, the "ssd_inception_v2_coco" and  the "faster_rcnn_inception_v2_coco".

#### Dataset

Our dataset used for the training of the classifier consists of:
- 280 images from the Udacity Simulator
- 710 images from the training bag that was recorded on the Udacity self-driving car

All images of the dataset where labeled manually. The dataset had to be converted into the TFRecord format.

#### Training
We have trained two classifier. One for the simulator and one for the real world. The training was done using the AWS Deep Learning AMI and the following configuration parameters:

- num_classes: 4 ('Green','Red','Yellow','Unknown')
- num_steps: 10000
- max_detections_per_class: 10

Other parameters were used from the sample configuration files provided by the tensorflow team (https://github.com/tensorflow/models/tree/master/research/object_detection/samples/configs).

#### Validation
We have then validated the trained classifier on 20 "unseen" images. Both classifier (for simulator and real world) have correctly detected and classified all traffic lights on the test images. Here are two examples:

<img src="./assets/light_classification_sim.png" width="400" height="300">
<img src="./assets/light_classification_real.png" width="400" height="300">

## Installation
### Native Installation

* Be sure that your workstation is running Ubuntu 16.04 Xenial Xerus or Ubuntu 14.04 Trusty Tahir. \[Ubuntu downloads can be found here\](https://www.ubuntu.com/download/desktop).
* If using a Virtual Machine to install Ubuntu, use the following configuration as minimum:
  * 2 CPU
  * 2 GB system memory
  * 25 GB of free hard drive space

  The Udacity provided virtual machine has ROS and Dataspeed DBW already installed, so you can skip the next two steps if you are using this.

* Follow these instructions to install ROS
  * \[ROS Kinetic\](http://wiki.ros.org/kinetic/Installation/Ubuntu) if you have Ubuntu 16.04.
  * \[ROS Indigo\](http://wiki.ros.org/indigo/Installation/Ubuntu) if you have Ubuntu 14.04.
* \[Dataspeed DBW\](https://bitbucket.org/DataspeedInc/dbw\_mkz\_ros)
  \* Use this option to install the SDK on a workstation that already has ROS installed: \[One Line SDK Install (binary)\](https://bitbucket.org/DataspeedInc/dbw\_mkz\_ros/src/81e63fcc335d7b64139d7482017d6a97b405e250/ROS_SETUP.md?fileviewer=file-view-default)
\* Download the \[Udacity Simulator\](https://github.com/udacity/CarND-Capstone/releases).

### Docker Installation
\[Install Docker\](https://docs.docker.com/engine/installation/)

Build the docker container
```bash
docker build . -t capstone
```

Run the docker file
```bash
docker run -p 4567:4567 -v $PWD:/capstone -v /tmp/log:/root/.ros/ --rm -it capstone
```

### Port Forwarding
To set up port forwarding, please refer to the \[instructions from term 2\](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/16cf4a78-4fc7-49e1-8621-3450ca938b77)

### Usage

1\. Clone the project repository

git clone https://github.com/camigord/System-Integration-Project

2\. Install python dependencies
```bash
cd CarND-Capstone
pip install -r requirements.txt
```
3\. Build project
```bash
cd ros
catkin_make
source devel/setup.sh
```

### Running

#### Simulator

We have developed two classifiers, the first one is a SSD, the second a RCNN. Simulation works well with both of them.
* To launch project with SSD classifier, type the following command. This is also used if no model parameter is specified or if the original launch file (that doesn't specify this parameter) is used.
```bash
roslaunch launch/styx.launch model:='frozen_inference_graph_simulation_ssd.pb'
```
* To launch project with RCNN classifier, type the following command:
 ```bash
roslaunch launch/styx.launch model:='frozen_inference_graph_simulation_rcnn.pb'
```
Then launch the simulator.

#### Real world testing

1\. Download \[training bag\](https://s3-us-west-1.amazonaws.com/udacity-selfdrivingcar/traffic\_light\_bag_file.zip) that was recorded on the Udacity self-driving car.
2\. Unzip the file
```bash
unzip traffic\_light\_bag_file.zip
```
3\. Play the bag file
```bash
rosbag play -l traffic\_light\_bag\_file/traffic\_light_training.bag
```
4\. Launch project in site mode
```bash
cd CarND-Capstone/ros
roslaunch launch/site.launch model:='frozen_inference_graph_real.pb'
```
