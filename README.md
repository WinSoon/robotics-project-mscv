# Université de Bourgogne

Robotics Engineering - ROS Project

## Report

**Authors**

Hong Win SOON, Theo PETITJEAN, Fabio RIBERO, Kumar ANKUSH

**Supervisors**

Ralph SEULIN, David FOFI, Raphael DUVERNE, Marc BLANCHON, Thibault CLAMENS


## Table of Contents
- [Introduction](#introduction)
  - [Overview](#overview)
- [Objectives](#objectives)


# Introduction

ROS or Robot Operating System is a meta-operating system used to operate robots or robotic hardware. A meta-operating system can be understood as an abstraction layer over an existing operating system and in that it provides us with a framework to test and design our robot.

It has a range of supported programming languages namely, C++ and Python and is a popular among the open source community. As such, there are a wide range of packages and codes that are available to us to choose from as well as extensive documentation on its operation. Utilising the freely available in-built and additional packages, ROS can give finer control of the robot without writing all control methods from scratch.


### Overview

The purpose of this project is to familiarize with ROS. Using unique packages pre-built by developers of ROS, we perform multiple actions of movement, localization, map building and then autonomous navigation. We will break down the process and discuss them in details further below.

# Objectives

The objectives in this project are enumerated below:

**1. Initialization and Navigation of a robot, Turtlebot 2 in this case**

For initialization and navigation of the robot, we used _turtlebot\_vibot_ package compiled by the Robotics Lab, France team at Condorcet (https://github.com/roboticslab-fr), University of Burgundy.


**2. Construction of a local map of the environment of the robot**

Using the same package we build the map of the robotics lab of Condorcet. 

**3. Movement of robot in the newly built map**

This step uses 'Rviz' to move the robot in the custom built map, this time within the confined boundaries without manual control.

**4. Defining the position points in the map for autonomous navigation**

Since the newly built map is a planar map, landmark position points in the map can be denoted by 2D vector.

**5. Using packages for reading QR code at the position points**

We used Python libraries for reading the QR codes at landmark points using the Kinect RGB camera.

**6. Play sound once QR code is detected**

For playing the sound upon successful registering of the QR code using another package called _sound-publisher_.

**7. Creating own launch files to unify all the functions**

Writing script for one launch file in order to launch all necessary scripts at once instead of calling them once at a time.

# Methodology

Let's break down certain important concepts of ROS and how it works.

- [_NODE_](http://wiki.ros.org/Nodes) : A node can be understood as a task, just like processes in an operating system. A node is a program to execute certain functions. By subscribing to a 'topic' and processing the information, a node then publishes the processed information to another 'topic'. 

- [_TOPIC_](http://wiki.ros.org/Topics) : A topic is a processed information index which has certain messages inside post- or pre-processing of information from a node. One can check existing topics by using command _rostopic_. 

- [_MESSAGES_](http://wiki.ros.org/Messages) : The information inside the various topics. Each topic, depending on the type of topic, has different information and may continuously publish information to the system.

- [_MASTER_](http://wiki.ros.org/Master) : It us the 'master' node. A system has unique master nodes for different devices. If an electronic system is interacting with different components which were not designed to work together natively, those components will have different master nodes which will publish processed information over a topic which can be subscribed by another master node.

- [_ROSLAUNCH_](http://wiki.ros.org/roslaunch) : It is the command to launch ROS commands which will in turn start the different nodes.

- [_RQT\_GRAPH_](http://wiki.ros.org/rqt_graph) : A visual representation of the nodes and topics of the system with connections showing where the nodes originate and which topics are being published and subscribed by which systems.

- [_ROSOUT_](http://wiki.ros.org/rosout) : Console log reporting system in ROS.


The packages we have used in our project are:
 - [*turtlebot\_vibot*](https://github.com/roboticslab-fr) - 
 Contains nodes which can initialize and operate the Kobuki base of the Turtlebot, the Kinect and the LIDAR connected to a computer. The package also has nodes for controlling the Kobuki base for navigation and publishing the odometry data with relevant topics in the subpackage _turtlebot\_vibot\_nav_. This subpackage also contains amcl (Adaptive Monte Carlo Localization) node to build a map with localization data which is fetched from LIDAR and Kinect. 

 - [*rbx1*](https://github.com/pirobot/rbx1) - 
 ROS By Example or rbx, contains multiple nodes which make use of various control nodes on both the master and the client terminal. We exclusively make use of a Python script in the subpackage _cv\_bridge\_demo.py_ in '_rbx1\_vision_' which we have modified to add our script to detect the QR code from Kinect's camera using OpenCV libraries.

- [*rplidar_ros*](https://github.com/Slamtec/rplidar_ros) - 

A Lidar package from Slamtec, installation and configuration of it is required for the operation of the Lidar

- [*cv_bridge*](http://wiki.ros.org/cv_bridge) -

Converts between ROS Image messages and OpenCV images. As the QR code reader program is based on the OpenCV platform, the bridge is required to translate between ROS and the OpenCV program.  

 - [custom package]()

The master and client are assigned their own WiFi modules, connected over a local network and a given static IPs. Usually, the client cannot be connected to two network adaptors at the same time. Once the client and the master have been turned on and connected over the local network, we run execute the custom launch file which in turn calls for execution of other launch files from the _turtlebot\_vibot_ 


# Mapping

The mapping was conducted with the reference guide provided from the lecturer [*turtlebot\_vibot*](https://github.com/roboticslab-fr/turtlebot_vibot).

Utilising Lidar, and manual control of the turtlebot through the use of the joystick, we bring the turtlebot around the lab in order to map out the lab.

We first perform the minimum launch on the turtlebot
```
$ roslaunch turtlebot_vibot_bringup minimal_rplidar.launch
```
followed by initialising the modofied gmapping demo file 

```
$ roslaunch turtlebot_vibot_nav gmapping_demo_rplidar.launch
```

Next we need to initialise the joystick teleop for the logitech controller on the Workstation
```
$roslaunch turtlebot_teleop logitech.launch --screen
```

As well as the Rviz on the workstation for visualization
```
$ roslaunch turtlebot_rviz_launchers view_navigation.launch
```
After moving carefully behind the turtlebot, we are able to obtain the following map 

![alt test](https://github.com/WinSoon/robotics-project-mscv/blob/master/img/map.JPG)

Which is saved with the following command 
```
$ roscd turtlebot_vibot_nav/maps/

$ rosrun map_server map_saver -f my_map
```

# Localization and  Navigation via the RVIZ

Before we can proceed with the navigation of the turtlebot, we need to first localize the map and then run the robot in the RVIZ manually to obtain the point coordinates and quartenion of the map. 

We first define the turtlebot mapfile variable and launch the AMCL demo file together using the following command 

```
$ roslaunch turtlebot_vibot_nav amcl_demo_rplidar.launch map_file:=`rospack find turtlebot_vibot_nav`/maps/Soon_map_15Nov.yaml
```
While we could also export the turtlebot mapfile variable using the following command, 

```
$ export TURTLEBOT_MAP_FILE=/...path.../map.yaml
```
We noticed that sometimes the command will result in a corrupted map, causing failure during the navigation process. 

We then run the RVIZ on the workstation 

```
$ roslaunch turtlebot_rviz_launchers view_navigation.launch --screen
```

In order to start the navigation process of the turtlebot through the RVIZ, we have to select the "2D point pose estimate" button to help the turtlebot discover its initial position. Finally, we can move the turtlebot around the map using the "2D Nav Goal" button. 

By moving the robot across the map and marking its initialisation position, we are able to obtain the quartenion information of the robot. This will allow us to specify the waypoints of the robot for later use during our autonomous navigation of the turtlebot. 

![alt test](https://github.com/WinSoon/robotics-project-mscv/blob/master/img/Quart_pose.jpg)

We then select 4 waypoints on the map to be used during the autonomous navigation process of the turtlebot. 

![alt test](https://github.com/WinSoon/robotics-project-mscv/blob/master/img/waypoint.JPG)

1) Home- Where we start the turtlebot and the place where the turtlebot returns 
2) Greek- The first waypoint, named in respect of the Greek team in our class. 
3) Front- The second way point, which is in front of the lab 
4) France- The third waypoint, named in respect of the French team in our class.
5) Rear- The fourth way point, which is at the back of the lab 


# Localization, Autonomous Navigation, and Task Activation

The core of the program is based on the ROS by Example code that is given to us in the ROS By Example Vol1, that can be obtained via this link 

https://github.com/pirobot/rbx1/blob/indigo-devel/rbx1_nav/nodes/nav_test.py

The program has been modified in order to suit and parts of the program is explained as follows: 
```
class NavTest():
    def __init__(self):
        rospy.on_shutdown(self.shutdown)

        # How long in seconds should the robot pause at each location?
        self.rest_time = rospy.get_param("~rest_time", 10)

```
We set the time for how long the robot will stop at each location. The reason why this is important is to give enough time for the robot to read the code and play the song. 

```
locations = dict()

        locations['Greek'] = Pose(Point(1.767, -0.075, 0.000), Quaternion(0.000, 0.000, 0.694, 0.720))
        locations['Front'] = Pose(Point(4.991, -2.083, 0.000), Quaternion(0.000, 0.000, -0.003, 1.000))
        locations['France'] = Pose(Point(1.973, -3.711, 0.000), Quaternion(0.000, 0.000, -0.708, 0.706))
        locations['Rear'] = Pose(Point(-1.036,-1.389, 0.000), Quaternion(0.000, 0.000, 1.000,0.013 ))
        locations['Home'] = Pose(Point(-0.038, 0.039, 0.000), Quaternion(0.000, 0.000, 0.690, 0.724))
```

The waypoints are then built into a dictionary, specifying the coordinates as well as the Quaternion pose of the robot. 

```
   # Subscribe to the move_base action server
        self.move_base = actionlib.SimpleActionClient("move_base", MoveBaseAction)

        self.bridgeDemo = qr_read.cvBridgeDemo()

        rospy.loginfo("Waiting for move_base action server...")
```
We then subscribe to the move base action server along with the QR code reader information. 

```
# Begin the main loop and run through a sequence of locations
        while not rospy.is_shutdown():
            # If we've gone through the current sequence,
            # start with a new random sequence
            if i == n_locations:
                i = 0
                sequence = ["Greek", "Front", "France", "Rear", "Home"]
                # Skip over first location if it is the same as
                # the last location
                if sequence[0] == last_location:
                    i = 1

            # Get the next location in the current sequence
            location = sequence[i]
```
Here we define the sequence in which the Turtlebot will move from one point to another. The turtle bot is set up so that it will move from the Home>Greek>Front>France>Rear before finally returning to its home position. After which it will repeat the whole process again. 

```
if not finished_within_time:
                self.move_base.cancel_goal()
                rospy.loginfo("Timed out achieving goal")
            else:
                state = self.move_base.get_state()
                if state == GoalStatus.SUCCEEDED:
                    rospy.loginfo("Goal succeeded!")
                    n_successes += 1
                    distance_traveled += distance
                    rospy.loginfo("State:" + str(state))
                    self.bridgeDemo.talk()
                else:
                  rospy.loginfo("Goal failed with error code: " + str(goal_states[state]))
```
Here we begin the reading process each time the robot has successfully reached its position. When the turtlebot sucessfully reaches a targetted waypoint, it will call out to the QR reader module, read the code that is displayed by the QR code, which will then be displayed in the form of a string. This in turn will then activate the song player to play a song according to the code that it has read.

If the robot has failed to reach its goal for whatever reason, it will print out an error message. 

The turtlebot will continue to go around the waypoints, until the program is terminated. 




