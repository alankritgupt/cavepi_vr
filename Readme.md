# VR Digital Twin of CavePI AUV in Meta Quest 2

## Compatible Platform
The packages are tested on Windows 11 computer and Meta Quest 2, and developed on ROS Noetic with Gazebo Classic and Unity (Editor version 6000.1.7f1).

## Prerequisites
1. First install the Meta Quest Link app for Meta Quest 2 and connect your Quest 2 with the Windows computer using the app.
2. Install Unity Hub and then install the editor version 6000.1.7f1 for the VR simulation project.
We will use Windows Subsystems for Linux (WSL) to run ROS and Gazebo in a Windows computer seamlessly.

## WSL Installation
1. Open Powershell and run as administrator.
2. Install WSL.
   ```sh
   wsl --install
   ```
    This will install the laterst version of Ubuntu with it. But we require Ubuntu 20.04 for this project.
3. Install Ubuntu 20.04 in WSL. In the same powershell window, run
   ```sh
   wsl --install -d Ubuntu-20.04
   wsl --update
   wsl --shutdown
   ```
4. Enable systemd inside WSL as required by ROS tools.
   ```sh
   sudo tee /etc/wsl.conf <<'EOF'
   [boot]
   systemd=true
   EOF
   ```
5. Now install ROS Noetic.
   
## ROS Noetic Installation with Gazebo Classic (Gazebo 11)
1. Setup ROS repositories and key.
   ```sh
   sudo apt update && sudo apt install -y curl gnupg lsb-release
   sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | \
        sudo gpg --dearmor -o /usr/share/keyrings/ros-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
     http://packages.ros.org/ros/ubuntu $(lsb_release -cs) main" | \
     sudo tee /etc/apt/sources.list.d/ros1.list
   ```
2. Install full desktop meta-package of ROS Noetic with Gazbeo Classic.
   ```sh
   sudo apt update
   sudo apt install -y ros-noetic-desktop-full
   ```
3. Install the `rosdep` package.
   ```sh
   sudo apt install -y python3-rosdep2
   ```
   Note: On some installs it's called `python3-rosdep`; if above fails, try `sudo apt install -y python3-rosdep`.
4. Initialize `rosdep` and set up the shell.
   ```sh
   sudo rosdep init
   rosdep update
   ```
   If this step gives an error as below:
   ```sh
   ERROR: default sources list file already exists:
        /etc/ros/rosdep/sources.list.d/20-default.list
   ```
   Re-initialize `rosdep`.
   ```sh
   sudo rm /etc/ros/rosdep/sources.list.d/20-default.list
   sudo rosdep init
   rosdep update
   ```
6. Source ROS setup file and add it to the `bashrc` file.
   ```sh
   echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
   source ~/.bashrc
   ```
7. Start and check if the Gazebo window appears without any errors.
   ```sh
   gazbeo
   ```

## Gazebo Simulation Setup
1. Download the `src` package for the ROS Noetic workspace of the Gazebo simulations from the link below at the Desktop of your Windows computer.
2. Create a folder named `Downloads`.
   ```sh
   mkdir Downloads
   ```
3. Copy the files from your Windows Desktop to the Downloads folder of Ubuntu in WSL.
   ```sh
   cp /mnt/c/Users/<WindowsUser>/Desktop/src.zip ~/Downloads/
   ```
4. Unzip the folder and copy it to a ROS workspace (say named 'cavepi_ws').
   ```sh
   sudo apt install -y zip
   sudo apt install -y unzip
   mkdir -p ~/cavepi_ws/src
   unzip ~/Downloads/src.zip -d ~/cavepi_ws/src
   ```
5. Now if you see an `src` folder inside `src` folder of `cavepi_ws`, bring that up in order.
   ```sh
   cd ~/cavepi_ws/src
   mv src/* .
   ```
   Now, the folder should be like `cavepi_ws > src >` all files.
6. Install the additional packages.
   ```sh
   sudo apt update
   sudo apt install -y bash-completion ros-noetic-rosbash
   sudo apt install -y ros-noetic-xacro
   sudo apt install -y ros-noetic-robot-state-publisher ros-noetic-image-view
   sudo apt install -y ros-noetic-joint-state-publisher
   sudo apt install -y ros-noetic-gazebo-ros-pkgs ros-noetic-gazebo-ros-control
   sudo apt install ros-noetic-mavros ros-noetic-mavros-extras
   sudo apt install ros-noetic-teleop-twist-keyboard
   ```
   In case if you are using any other version of ROS than Noetic, please replace `noetic` with the correct version of ROS.
7. Install the `scipy` package with version >= 1.6.
   ```sh
   sudo apt install python3-pip
   pip3 install --user --upgrade scipy
   ```
8. Now build the workspace and run the launch file.
   ```sh
   cd ~/cavepi_ws
   catkin_make
   source devel/setup.bash
   roslaunch nemogator_bringup cavepi_auv_in_a_cave.launch
   ```
   A Gazebo window will pop up and starting the simulation would show three windows of different views of the robot.
   
## Connecting Gazebo with Unity for VR Visualisation
1. Turn on the Meta Quest 2 and connect it with Meta Quest Link app either on WiFi or wired connection.
2. Download the Unity project named `My project` from [Dropbox link](https://www.dropbox.com/scl/fi/ep7kiitvje05xrhtthl2p/My-project.zip?rlkey=a7kvok0bcftymopx4gk3evaf4&st=8nhn81de&dl=0). Open the project in Unity Editor version 6000.1.7f1.
3. Start the Gazebo simulation in one Powershell window.
4. Open other Powershell window as administrator and run the following:
   ```sh
   sudo apt install ros-noetic-rosbridge-server
   roslaunch rosbridge_server rosbridge_websocket.launch
   ```
5. Put on the headset and start the simulation from Unity Editor. Have fun watching your VR simulation now!
