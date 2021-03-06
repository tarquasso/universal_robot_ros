# universal_robot_ros
OMPL Motion planner communicating with Universal Robot UR5

## Setup UR5 with ROS

1. Install ROS Kinetic on Ubuntu 16.04
http://wiki.ros.org/kinetic/Installation/Ubuntu

    1. Setup Ubuntu Repos
    Configure your Ubuntu repositories to allow "restricted," "universe," and "multiverse." You can follow the Ubuntu guide for instructions on doing this. Open "Software & Updates" and make sure that on the first tab of this window, all three are checked on.

    1. Sources.list
`sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

    1. Keys
`sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116`

    1. Installation
`sudo apt-get update`
Desktop-Full Install (Recommended): ROS, rqt, rviz, robot-generic libraries, 2D/3D simulators, navigation and 2D/3D perception
`sudo apt-get install ros-kinetic-desktop-full`

    1. Init rosdep
Before you can use ROS, you will need to initialize rosdep. rosdep enables you to easily install system dependencies for source you want to compile and is required to run some core components in ROS.
```
sudo rosdep init
rosdep update
```

    1. Environment Setup
```
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

    1. Dependencies for building packages
`sudo apt-get install python-rosinstall python-rosinstall-generator python-wstool build-essential`

2. Test ROS Installation:
http://wiki.ros.org/ROS/Tutorials

Installing and Configuring according to:
http://wiki.ros.org/ROS/Tutorials/InstallingandConfiguringROSEnvironment

Check of environment variables are set up:
`printenv | grep ROS`
Should show  ROS_ROOT and ROS_PACKAGE_PATH

3. Create a ROS workspace:
```
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/
catkin_make
```

add and source new setup.*sh file:
```
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```
Make  sure workspace is properly overlayed by running:

```
echo $ROS_PACKAGE_PATH
```
It should show:
/home/youruser/catkin_ws/src:/opt/ros/kinetic/share

4. Navigate the ROS Filesystem to test
```
sudo apt-get install ros-kinetic-ros-tutorials
```

5. Install UR5 Packages:
http://wiki.ros.org/universal_robot/Tutorials/Getting%20Started%20with%20a%20Universal%20Robot%20and%20ROS-Industrial

First install ros industrial core:
http://wiki.ros.org/Industrial/Install
```
sudo apt-get install ros-kinetic-industrial-core
```

Then install universal_robot driver, because Robot description and MoveIt configurations from the universal_robot stack can still be used. Do the installation according to:
http://wiki.ros.org/universal_robot
by executing:
```
sudo apt-get install ros-kinetic-universal-robot
```

Next, clone and install ur_modern_driver according to:
https://github.com/ros-industrial/ur_modern_driver
For this, first clone repo into catkin_ws
```
cd catkin_ws/src
git clone git@github.com:ros-industrial/ur_modern_driver.git
```
Change to the branch that has the kinetic code for the ur modern driver:
```
cd ur_modern_driver
git checkout kinetic-devel
```
Then make the workspace
```
cd ~/catkin_ws
catkin_make
```
## Running the code

6. Running it now according to section: "3.5 Making contact with UR v3.x" at:
http://wiki.ros.org/universal_robot/Tutorials/Getting%20Started%20with%20a%20Universal%20Robot%20and%20ROS-Industrial 

Start UR5, then setup its IP address to be on the same subnet. For us, the IP becomes: 192.168.1.106. Next, initialize the robot with the proper payload.
Back to the linux machine, note that as MoveIt! seems to have difficulties with finding plans for the UR with full joint limits [-2pi, 2pi], there is a joint_limited version using joint limits restricted to [-pi,pi]. In order to use this joint limited version, simply use the launch file arguments 'limited'.

In shell #1, run:
```
roslaunch ur_modern_driver ur5_bringup.launch limited:=true robot_ip:=192.168.1.106 [reverse_port:=REVERSE_PORT]
```
Note: replace "192.168.1.106" with the IP of the robot, shich has to be on the same subnet as the linux machine.

For setting up the MoveIt! nodes to allow motion planning run (assumes the connection is already established from the other shell), in a new shell #2, run:
```
roslaunch ur5_moveit_config ur5_moveit_planning_execution.launch limited:=true
```

For starting up RViz with a configuration including the MoveIt! Motion Planning plugin, in a new shell #3, run:
```
roslaunch ur5_moveit_config moveit_rviz.launch config:=true
```
In RViz, select the second tab to plan and execute motions after you have dragged the end effector of the robot to a new desired pose.
