## License
This project is licensed under MIT License. 
For previous licensing history, see [LICENSE.previous.md](https://github.com/YJ0528/minibot/blob/main/LICENSE.previous.md)

<br>

<div align="center">
	<img src="https://github.com/YJ0528/minibot/blob/main/visual_demos/robot_visual.jpg" height="350">
	<br>
	<h2> Differential Drive Robot using ROS 2 Jazzy Jalisco </h2>
	<a href="https://github.com/YJ0528/minibot/blob/main/LICENSE">
		<img src="https://img.shields.io/static/v1.svg?label=License&message=MIT&color=blue&style=flat-square" height="20">
  	</a>
	<a href="https://ubuntu.com/tutorials/install-ubuntu-desktop#1-overview">
		<img src="https://img.shields.io/static/v1.svg?label=Ubuntu&message=24.04%20LTS&color=orange&style=flat-square" height="20">
  	</a>
	<a href="https://docs.ros.org/en/jazzy/index.html">
	    <img src="https://img.shields.io/static/v1.svg?label=ROS%202&message=Jazzy%20Jalisco&color=0059b3&style=flat-square" height="20">
  	</a>
	<br><br>		
</div>

Hi everyone! Today I would like to share my project implementing a fundamental differential drive robot using <ins>**ROS2 Jazzy Jalisco**</ins> and <ins>**Raspberry Pi 5**</ins>, with features including Gazebo simulation, ros2_control, teleoperation, SLAM, and Navigation2.

This project also represents my learning journey following tutorials from [Articulated Robotics](https://articulatedrobotics.xyz/):

*	Forum: https://articulatedrobotics.xyz/tutorials/mobile-robot/project-overview/
*	Tutorial playlist: https://youtube.com/playlist?list=PLunhqkrRNRhYAffV8JDiFOatQXuU-NnxT&si=1N9GNN6gRnet5heK

You can see the demo [here](https://github.com/YJ0528/minibot/blob/main/README.md#demo)

<br>	

## Getting Started
Before we start, make sure you are using [Ubuntu 24.04](https://ubuntu.com/tutorials/install-ubuntu-desktop#1-overview) and have [ROS 2 Jazzy Jalisco](https://docs.ros.org/en/jazzy/index.html) installed in your server machine (which is your desktop or laptop).

Make sure to install git:

	sudo apt install git ros-dev-tools -y

Make sure VScode is installed, it can be found in Ubuntu App Center. Some extensions that I have installed are listed below:
*	C/C++ Extension Pack (Microsoft)
*	Python (Microsoft)
*	ROS (Microsoft)
*	YAML (Red Hat)
*	Remote - SSH (Microsoft)

Additionally, consider to install [Terminator](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#terminator), which is a useful tool to use with ROS 2.

<br>

## Setting Up ROS 2 Workspace and Installed Requied Packages
Before we start, see [ROS 2 Packages: A Brief Introduction](https://github.com/YJ0528/minibot/blob/main/Package_Installation_Instruction.md#introduction) for the differences between the <ins>**binary installation**</ins> and <ins>**build from source**</ins>

### Creating a Workspace `ros_ws`
1.	Open a terminal using `ctrl + alt + t`, create a workspace named `ros_ws` with a folder named `src`:

		mkdir -p ~/ros_ws/src
	*	Replace `ros_ws` with the name you want to put as your workspace name.

3.	 Follow to the insturction at [**Install ROS 2 Packages**](https://github.com/YJ0528/minibot/blob/main/Package_Installation_Instruction.md#install-ros-2-package) to install all the necessary packages.
		*	Although only the listed items under **<ins>For simulation only</ins>** are required for simulation, it is recommanded to install all of them.

<br>

## Launch Simulation
1.	Once Everything is set, go to `your_workspace` and source the workspace:
	
	 	cd ros_ws/
		. install/setup.bash
	*	When a new terminal is opened, ensure that terminal sourced the workspace also:
	
2.	Launch the simulation:
	
		ros2 launch minibot sim.launch.py
	*	This will launch the `robot description`, `gazebo`, `rviz2`, `ros2_control` etc.
	
3.	Launch Simulation Control and SLAM

	The `sim_control_station.launch.py` will launch all control features for the simulation, inclduing `teleop`, `slam_toolbox`, `nav2 stack`.
	
	*	To run the control with `online_async_slam` from `slam_toolbox`, open a new terminal, source the local workspace and enter:
	
			ros2 launch minibot sim_control_station.launch.py use_slam_option:=online_async_slam
	*	For `mapper_params_localization` from `slam_toolbox`:
	
	 		ros2 launch minibot sim_control_station.launch.py use_slam_option:=mapper_params_localization
	 	*	To change the map to load, open the params file at [`./src/minibot/config/mapper_params_localization.yaml`](https://github.com/YJ0528/minibot/blob/aa18371856751b270af9280b53b87c7f5f3a6bcf/config/mapper_params_localization.yaml#L18), replace the value corresponding `map_file_name` with the desired directory.
	*	For `AMCL` from `nav2`:
	
			ros2 launch minibot sim_control_station.launch.py use_slam_option:=amcl map:=./src/minibot/maps/sample_map.yaml
	 	*	To change the map to load, replace the default value corresponding to [`map`](https://github.com/YJ0528/minibot/blob/aa18371856751b270af9280b53b87c7f5f3a6bcf/launch/sim_control_station.launch.py#L27) with the desired directory.

5.	Alternatively, we can just run the teleoperation only using:

		ros2 launch minibot joystick_teleop.launch.py
### Additional Notes:
Refer to:
*	[ros_gz_bridge](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#ros_gz_bridge) if a topic could not be send or recived between ROS 2 and Gazebo.
*	[How a ROS 2 Topic Is Received By a Node](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#how-a-ros-2-topic-is-received-by-a-node) if you suspect topics are not being published or subscribed to properly
*	[How to Check ROS 2 Node Parameter Name using Command Line](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#how-to-check-ros-2-node-parameter-name-using-command-line) if you want to check if the parameters from `.yaml` file is loaded to the node properly.

<br>
<br>


<div align="center">
	<h2> Setting Up the Robot </h2>
	<img src="https://github.com/YJ0528/minibot/blob/main/visual_demos/robot_hardware.jpg" height="500">
	<p>Robot Hardware and Circuit Connection Setup</p>
</div>

***<ins>IMPORTANT:</ins>** Please exercise the proper safety for lipo battery, I have included some precautions at [Lipo Battery Safety Precautions](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#lipo-battery-safety-precautions).

## Lisf of Hardwares:
*	Raspberry Pi 5
*	RPLIDAR C1
*	ESP32-vroom-32 development board
*	ESP32 expansion board (be award whether the pinout match the ESP32 dev board)
*	Cytron MDD3A motor driver
*	3S lipo battery
*	Step down converter ([DFRobot DRF0205](https://www.dfrobot.com/product-752.html) for example)

I only listed down the main electronics that I used, for more information, refer to [Build a Mobile Robot with ROS: Bill of Materials](https://articulatedrobotics.xyz/tutorials/mobile-robot/project-overview/#bill-of-materials) by [Articulated Robotics](https://articulatedrobotics.xyz/)

see also: [Recommanded Components for Wiring and Robot Chasis (Optional)](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#recommanded-components-for-wiring-and-robot-chasis-optional).

<br>

## Configuring the Raspberry Pi:
Make sure you are using [Ubuntu 24.04](https://ubuntu.com/tutorials/install-ubuntu-desktop#1-overview) and have [ROS 2 Jazzy Jalisco](https://docs.ros.org/en/jazzy/index.html) installed in the Raspberry Pi.

**<ins>Install Ubuntu in Raspberry PI using rpi-imager**</ins>
*	Make sure you have a Micro SD Card with at least 32GB, but a Micro SD with <ins>**64GB or more**</ins> is recommanded.
	*	The system has occupied about 24GB of my SD Card, including all everything needed to run the robot.
*	If you never install Ubuntu 24.04 using rpi-imager before, see: [RPI5: Flash Ubuntu to SD Card](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#rpi5-flash-ubuntu-to-sd-card).

**<ins>Install ROS 2 in Raspberry PI</ins>**	
*	For ROS 2 installation, you can select `ROS-Base Install` instead of `Desktop Install`.
*	If you wish to run `RViz` in Raspberry Pi, consider `Desktop Install` or binary install via bash using `sudo apt install ros-jazzy-rviz2`.

<br>

## Installing Packages for Raspberry Pi and Flashing Code to ESP32

<ins>**Creating a Workspace `robot_ws` in Your SBC (Raspberry Pi)**</ins>
1.	Open a terminal using `ctrl + alt + t`, create a workspace named `robot_ws` with a folder named `src`:

		mkdir -p ~/robot_ws/src
	*	Replace `robot_ws` with the name you want to put as your workspace name.

2.	 Follow to the insturction at [**Install ROS 2 Packages**](https://github.com/YJ0528/minibot/blob/main/Package_Installation_Instruction.md#install-ros-2-package) to install all the necessary packages in Raspberry Pi.
		*	You need to install all the packages listed except `ros_gz`.

<ins>**Flashing Code to ESP32**</ins>

In addition, you need to Flash the driver code to your ESP32, see [Install and Flash Microcontroller Driver code](https://github.com/YJ0528/minibot/blob/main/Package_Installation_Instruction.md#install-and-flash-microcontroller-driver-code)

<br>	

## Operating the Robot

1.	Ensure the Raspberry Pi USB device port number matched the value declared:

	*	`/dev/ttyUSB0` for the lidar serial; located at [`./src/minibot/launch/robot.launch.py/declare_lidar_serial_port`](https://github.com/YJ0528/minibot/blob/aa18371856751b270af9280b53b87c7f5f3a6bcf/launch/robot.launch.py#L53).
	*	`/dev/ttyUSB1` for the ESP32; located at [`./src/minibot/description/ros2_control.xacro/RobotSystem/device`](https://github.com/YJ0528/minibot/blob/aa18371856751b270af9280b53b87c7f5f3a6bcf/description/ros2_control.xacro#L11).
	*	To check or troubleshoot the USB connection in Raspberry Pi, see [RPI5: Add USB Access for Raspberry Pi](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#rpi5-add-usb-access-for-raspberry-pi).

2.	Connects to the Raspberry Pi from your server machine using `openssh-server`. Open a terminal in the server machine, enter:

		ssh <remote_username>@<remote_ip_address>
	 *	If you never install or use `openssh-server` before,see [SSH Access to Remote Machine- Connect Remote Machine via Terminal](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#connecting-to-the-remote-machine-in-terminal).  

3.	In order to let the nodes to be discoverable between the server and the remote machine, We need to set `ROS_DOMAIN_ID` to the same for both of the machine:

		# The ID is 0 by default, but can be any number between 0 between 101
		export ROS_DOMAIN_ID=1
	*	For the Raspberry Pi, we can add `export ROS_DOMAIN_ID = 1` to the `.bashrc` file instead using:

			echo "export ROS_DOMAIN_ID=1" >> ~/.bashrc
	*	To check the `ROS_DOMAIN_ID` set, enter `echo ${ROS_DOMAIN_ID}` in terminal.

4.	Once Everything is set, go to `your_workspace` and source the workspace:

		cd ros_ws/
		. install/setup.bash
	*	When a new terminal is opened, ensure that terminal sourced the workspace also:

5.	Run the robot:

		ros2 launch minibot robot.launch.py
	*	This will launch `robot description`,`ros2_control`, `sllidar_c1_launch` etc.

6.	Launch Robot Control with SLAM or Localization

	The `robot_remote_station.launch.py` will launch all control features for the robot, inclduing `teleop`, `slam_toolbox`, `nav2 stack`.
	
	*	To run the control with `online_async_slam` from `slam_toolbox`, open a new terminal in your server machine, source the local workspace and enter:
	
			ros2 launch minibot robot_remote_station.launch.py use_slam_option:=online_async_slam
	*	For `mapper_params_localization` from `slam_toolbox`:
	
	 		ros2 launch minibot robot_remote_station.launch.py use_slam_option:=mapper_params_localization
	 	*	To change the map to load, open the params file at [`./src/minibot/config/mapper_params_localization.yaml`](https://github.com/YJ0528/minibot/blob/aa18371856751b270af9280b53b87c7f5f3a6bcf/config/mapper_params_localization.yaml#L18), replace the value in `map_file_name` with the desired directory.
	*	For `AMCL` from `nav2`:
	
			ros2 launch minibot robot_remote_station.launch.py use_slam_option:=amcl map:=./src/minibot/maps/sample_map.yaml
	 	*	To change the map to load, replace the default value of [`map`](https://github.com/YJ0528/minibot/blob/aa18371856751b270af9280b53b87c7f5f3a6bcf/launch/robot_remote_station.launch.py#L27) with the desired directory.
7.	Alternatively, we can just run the teleoperation only using:

		ros2 launch minibot joystick_teleop.launch.py

<br>

## Demo:
Some visual demostration:
<div align="center">
	<h3> SLAM with Joystick Teleoperation (5x speed with defualt parameter) </h3>
	<img src="https://github.com/YJ0528/minibot/blob/main/visual_demos/SLAM_demostration-ezgif.com-video-to-gif-converter.gif" height="400">
	<br><br>	
 	<h3> Nav2 Navigation using SLAM Toolbox Localization (5x speed)</h3>
	<img src="https://github.com/YJ0528/minibot/blob/main/visual_demos/Nav2_demostration-ezgif.com-video-to-gif-converter.gif" height="400">	
</div>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`global_costmap/global_costmap/ros__parameters/inflation_layer/cost_scaling_factor`= 1.0

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`global_costmap/global_costmap/ros__parameters/inflation_layer/inflation_radius`= 0.05


## TO DO (maybe):
*    Implement the hardware interface within this package instead.
*    Explore the `slam_toolbox` and `nav2`, or any equivalent.
*    Implement 3D slam at least in simulation using depth camera, if possible.

