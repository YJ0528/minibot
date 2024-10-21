This file consists of some concept to handle ROS 2 batter and some implementation suggestion for reference and troubleshooting.
### List of Content:
*	[Explanation on ROS 2 Packages](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#explanation-on-ros-2-packages)
*	[How a ROS 2 Topic Is Received By a Node](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#how-a-ros-2-topic-is-received-by-a-node)
*	[How to Check ROS 2 Node Parameter Name using Command Line](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#how-to-check-ros-2-node-parameter-name-using-command-line)
*	[How to Increase Virtual Memory](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#how-to-increase-virtual-memory)
*	[Lipo Battery Safety Precautions](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#lipo-battery-safety-precautions)
*	[Recommanded Components for Wiring and Robot Chasis (Optional)](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#recommanded-components-for-wiring-and-robot-chasis-optional)
*	[ros_gz_bridge](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#ros_gz_bridge)
*	[RPI5: Add USB Access for Raspberry Pi](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#rpi5-add-usb-access-for-raspberry-pi)
*	[RPI5: Flash Ubuntu to SD Card](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#rpi5-flash-ubuntu-to-sd-card)
*	[SSH Access to Remote Machine](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#ssh-access-to-remote-machine)
*	[Terminator](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#terminator)
<br>

## Explanation on ROS 2 Packages
ROS 2 installed its packages mainly in 2 types of location, which recognize as workspace:

*	A <ins>**core workspace**</ins> located at `/opt/ros/<YOUR_ROS_DISTRO>`

*	<ins>**Local workspaces**</ins>, which can be create freely using:
	
		mkdir -p ~/<workspace_name>/src
The differences between <ins>core workspace</ins> and <ins>local workspaces</ins> are listed below:

| Feature | Core Workspace | Local workspaces | 
|-----|----|----|
| Role  			| contain all the essential libraries and packages.  					| space we use to develop our robot system; provide felxibility to experiment any packages we want.		| 
| Workspace layer		| <ins>Underlay workspace</ins>, all local workspace can access here when needed. 	| <ins>Overlay workspace</ins>, Content in local workspaces will not share between each other.			| 
| Package included 		| only official-released ROS 2 packages 						| mainly packages under development or third-party packages 							| 
| Workspace number(s)		| normally only one for each distro 							| can be multiple												| 
| Folder location(s)  		| located at `/opt/ros/<YOUR_ROS_DISTRO>` 						| can be create freely using mkdir `-p ~/<workspace_name>/src`							| 
| Installation method 		| <ins>Binary Installation</ins> using `sudo apt install ros-jazzy-<package_name>` 	| <ins>Build from source</ins> using `colcon build...` at the root of local workspace, see [here](https://docs.ros.org/en/jazzy/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html#tasks). 				| 
| Installation difficulty 	| simple, the system itself will choose the suitable version 				| can be more difficult, version incompatible could happen; some understanding towrads the package is required 	| 
| Package reliability  		| Reliability is guaranteed, tested before release 					| reliability might varies, some packages might not available for newer version.				| 
| Package priority 		| lower											| higher 													| 
| Documentation  		| Usually comes with official documentation 						| Mainly included in README.md 											| 
| Sourcing the workspace	| `source /opt/ros/jazzy/setup.bash `; add to shell startup scrip using: `echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc`	| In local's root workspace, `source install/setup.bash` 	| 
<br>

## How a ROS 2 Topic Is Received By a Node 
In ROS 2, for a topic to be received by a subscriber node, the following have to match each other:
*	**Topic name**
*	**Topic message type**, defines the message format and content
A topic can have multiple message type, but the value will not share between the message types.

### Manually Identify the Message Type in ROS 2 Core Package.
We can imagine topic message type as the <ins>**directory towards the hearder file**</ins> that included all the necessary implementation information (counting from `/opt/ros/jazzy/include` onwards).

To check the message type, we can:
1.	Locate to `/opt/ros/jazzy/include`
2.	**Most (not all)** message type will suffixed with `_msgs` at the first directory, or `msg` in the subsequent path.
3.	For example,topic cmd_vel has a message type `geometry_msgs/msg/TwistStamped`, hence its header file is located at `/opt/ros/jazzy/include/geometry_msgs/msg`

### Checking the Topic and Topic Message Type using Command Line
In the terminal:

To list all available ROS 2 topics:

	ros2 topic list
To check if ROS 2 topic is publishing message:

	ros2 topic echo /topic_name
To check ROS 2 topic message type:

	ros2 topic type /topic_name
To list all available gazebo topics:

	gz topic -l
To check if gazebo topic is publishing message:

	gz topic -t /topic_name -i
To check gazebo topic message type:

	gz topic -e /topic_name
<br>

## How to Check ROS 2 Node Parameter Name using Command Line

1.	launch the node (e.g. twist_mux node) using configuration in .yaml file:
	
		##Example:
		ros2 run twist_mux twist_mux --params-file src/minibot/config/joystick_params.yaml
2.	In another terminal, list out all availabe nodes to check the name:

   		ros2 node list
	which return:

		/twist_mux
3.	Check the paramter of the node /twist_mux using:

		ros2 param dump /twist_mux
	which return (for example):

		twist_mux:
		  ros__parameters:
		    diagnostic_updater:
		      period: 1.0
		      use_fqn: false
		    output_stamped: true
		    qos_overrides:
		      /parameter_events:
		        publisher:
		          depth: 1000
		          durability: volatile
		          history: keep_last
		          reliability: reliable
		    start_type_description_service: true
		    use_sim_time: false
	*	In this case, the parameter name to publish stamped cmd_vel is `/output_stamped`
<br>

## How to Increase Virtual Memory
Some packages such as Gazebo or Moveit2 required decent amount of memory when building the packages from source, having an insufficient RAM(e.g. 8GB) may risk the machine to stop responding and subsequently fail the build. 

Such risk can be reduced by increasing the virtual memory. In Ubuntu, virtual memory is stored in term of swap space. 

Following method to increase the swap space is extracted from [askUbuntu: How to increase swap space?](https://askubuntu.com/questions/178712/how-to-increase-swap-space).

1.	Firstly, open the terminal and input the following: 

		dd if=/dev/zero of=/media/fasthdd/swapfile.img bs=1024 count=1M
	*	This will create a swap file containing 1GiB virtual memory named `swapfile.img`
	*	To change the swap size to 4GiB, replace `count=1M` to `count=4M`
2.	Bake the swap file using:

		mkswap /media/fasthdd/swapfile.img
3.	To activate the swap space during boot, add the following to `dev/fstab`:

		/media/fasthdd/swapfile.img swap swap sw 0 0
4.	Reboot or enter the following command in terminal to activate the swap space:

 		swapon /media/fasthdd/swapfile.img
5.	To check if the swap space is available:

		cat /proc/swaps
	*	If the swap space is avaialble, you should see the name of the swapfile and the size in `byte`.

<br>

## Lipo Battery Safety Precautions
Lipo battery is highly volatile especially due to its high discharge rate. It can causes a fire if being handled properly.

*** **IMPORTANT** *** **Please do not hesitate to spend money on the accessories.** 
*	Do not leave the charging battery unsupervised.
*	Get a battery charger with **balance charge** and **storage charge**.
*	Balance charge if possible.
*	Consider getting a [bat-safe charging box](https://hobbyking.com/en_us/bat-safe-lipo-battery-charging-safe-box.html?utm_source=youtube&utm_medium=social&utm_campaign=woolysheep_032021&___store=en_us) or a lipo bag at least.
*	Ensure no combustable material nearby.
*	Storage charge the battery asap once received, especially when not planning to use it immediately.
*	Discharge (or use) a charged battery within 3 days, possibly use it the next day.
*	Keep track of each of the battery voltage using battery checker.

### Imbalanced Voltage Value Across Batteries
Given a 3S battery for example, the battery consists of 3 serially-connected lipo batteries, with a voltage reading between 3.20V to 4.20V for each of them. Normally each of the batteries will have a voltage value that is close to each other.

An imbalance voltage value across each of the batteries could indicate an its unhealthy state, and should be aware of since the battery voltage could drop below the low limit (3.20V) although the total voltage value is still above the limit (3.20 x3 = 9.60V, but also 3.00 + 3.30 x 2 = 9.60V), resulting a one-sided puffing battry for example. 

If the voltage imbalance is observed, check the battery voltage more frequently using a bettery checker when discharging the battery. 

<br>

## Recommanded Components for Wiring and Robot Chasis (Optional)
Item listed below are some recommandation for building the wire connection and the robot chasis: 
*	Wire ferrule/ double wire ferrule
*	XT60 connector or T-plug to connect the lipo battery
*	Terminal strips
*	Terminal block
*	USB to terminal block connector
*	3.6v/4.8V screwdriver
*	12V screwdriver for drilling
*	hot air gun
*	multimeter
*	wire crimper
*	heat shrink tube
*	molex connecter and pin
*	screw set

<br>

## ros_gz_bridge
Starting from ROS 2 Jazzy, plugins are no longer supported in connecting gazebo and ROS 2 topics. Instead ros_gz_bridge is used for this purpose, see: [Use ROS 2 to interact with Gazebo](https://gazebosim.org/docs/harmonic/ros2_integration/#use-ros-2-to-interact-with-gazebo).

An example implementation in launch file are as follow:

	node_gz_bridge = Node(
	        package='ros_gz_bridge',
	        executable='parameter_bridge',
	        arguments=[	
	            # Gazebo_Control
	            '/cmd_vel@geometry_msgs/msg/Twist]gz.msgs.Twist',
	        ],
	        output='screen'
	    )
*	This will send the topic message named cmd_vel from ROS 2 to gazebo.
*	Replace `]` with `@` to established a connection in both direction, and `[` to send topic message from Gazebo to ROS.

### ROS-Gazebo message type conversion
**In most case (not all)**, gazebo message type will be `gz.msg.<the last element of ROS 2 message type>`.

Given the topic `cmd_vel` type name in ROS 2 is `/cmd_vel@geometry_msgs/msg/Twist`, its message type in gazebo will be `gz.msgs.Twist`

To retrive the values directly from command line, see: [Checking the Topic and Topic Message Type using Command Line](https://github.com/YJ0528/minibot/blob/main/Tips_and_Troubleshooting.md#checking-the-topic-and-topic-message-type-using-command-line) 

<br>

## RPI5: Add USB Access for Raspberry Pi
First, we check if the serial port is found in /dev using:

    ls -l /dev/ttyUSB*
  *  If serial port is conencted it should return `/dev/ttyUSB0`, `/dev/ttyUSB1`...

If the issue presist, it could indicate that raspberry pi is preventing the user from accessing the device. 

To resolve this:

**A.  Add user to diaout group:**
        
    sudo usermod -a -G dialout $USER
  *  Logout and login again for the change to take effect.
     
**B.  Create a udev rule:**

*** **IMPORTANT** *** **I got this fix from Claude and logically I did not actually understand how it works, so it might not work as expected, please implement with caution.**

1.  Create a udev rule:

        sudo nano /etc/udev/rules.d/99-sllidar.rules
2.  Add the following line to the file:

        # SLLIDAR rule
        SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", ATTRS{serial}=="4466c1d8fee5ed118b4bd0a80b2af5ab", SYMLINK+="ttyLIDAR", MODE="0666"
        
        # ESP32 rule
        SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", ATTRS{serial}=="0001", SYMLINK+="ttyESP32", MODE="0666"
        
        # Rule to ignore both devices in ModemManager
        SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", ENV{ID_MM_DEVICE_IGNORE}="1"
      *  exit `ctrl + x` and save (press `y` followed by `enter` ) the file using command: 
4.  Reload the udev rules:

        sudo udevadm control --reload-rules && sudo udevadm trigger
<br>

## RPI5: Flash Ubuntu to SD Card

### Note:
As of the lastest update according to the 
[Ubuntu MATE 24.04 LTS Release Notes](https://ubuntu-mate.org/blog/ubuntu-mate-noble-numbat-release-notes/), and 
[Ubuntu MATE 24.10 Release Notes](https://ubuntu-mate.org/blog/ubuntu-mate-oracular-oriole-release-notes/),
the Ubuntu mate is not available for Raspberry Pi yet. Hence, this section will install `Ubuntu desktop 24.04` instead.

### Instruction
1.	Install rpi-imager from Ubuntu App Center.
	*	Alternatively, install rpi-imager from bash:

			sudo apt update && sudo apt install rpi-imager

2.	Insert your SD card to the computer, open the imager, choose the Raspberry Pi device that your are using.
3.	Select the desired OS.
	*	For Ubuntu OS, it should located at `Other general-purpose OS/Ubuntu`, select the desktop version.
4. 	Select your SD card as the storage and flash the OS.
5. 	Once done, insert the flashed SD card into the Raspberry Pi and the it is ready to run.

<br>

## SSH Access to Remote Machine
### Connect Remote Machine via Terminal
OpenSSH server allows our computer to remotely control networked machine such as Raspberry Pi like a server.

<ins>**To Connect to remote using OpenSSH server:**</ins>
1.	Install OpenSSH server:

		sudo apt install openssh-server
2.	Connect the Raspberry Pi to a monitor, open a terminal and enter:

		whoami
		ip addr
	*	`whoami`: will return your remote machine's `username` 
	*	`ip addr`: will return a long chain of information, including something like `inet 192.168.1.100/24`, in which the ip address will be `192.168.1.100`

3.	In our server machine, or a computer, open a terminal and enter:

		ssh <remote_username>@<remote_ip_address>
 ### VScode Remote - SSH extension
 The extension allows the dev machine access the remote machine's files and folder in VScode. To set up the connection, see: [ Connect to a Remote Server with SSH in VS Code -- Step-by-Step Tutorial ](https://www.youtube.com/watch?v=qZwGyfN8X5A).
<br>	

## Terminator
Terminator allows multiple terminal to be open in the same window, see: [Terminator](https://innovativeinnovation.github.io/ubuntu-setup/terminals/terminator.html)

<div align="center">
	<img src="https://innovativeinnovation.github.io/ubuntu-setup/images/terminator.png" height="400">
	<br>
	source: <a href="https://innovativeinnovation.github.io/ubuntu-setup/terminals/terminator.html">Terminator </a>
	<br><br>		
</div>

To install Terminator in Ubuntu:
<br>
<br>

 	sudo apt install terminator
Shortcut keys:
*	`ctrl + o`: Open an new terminal below the current terminal. 
*	`ctrl + e`: Open an new terminal at the right of the current terminal. 
*	`ctrl + shift + t`: Open an new tab.
*	`ctrl + w`: Close the current terminal.
*	`ctrl + shift + arrow_key`: Move current terminal border.

