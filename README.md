# AVP-SLAM++
**Authors**: [Vivek Jaiswal](mailto:vjaiswal@umich.edu), [Harsh Jhaveri](mailto:hjhaveri@umich.edu), [Jack Lee](mailto:chunhlee@umich.edu), [Devin McCulley](mailto:devmccu@umich.edu)

AVP-SLAM++ is an extension on the AVP-SLAM-PLUS repository initially implemented by [Liu Guitao](mailto:liuguitao@sia.cn). 

AVP-SLAM-PLUS is an implementation of [AVP-SLAM: Semantic Visual Mapping and Localization for Autonomous Vehicles in the Parking Lot (IROS 2020)](https://arxiv.org/abs/2007.01813) with some new contributions including:
* The addition of a multi-RGBD camera mode. AVP-SLAM was initially only implmented with multiple RGB cameras
* The addition of using normal distribution transformation (NDT) for localization. As published, AVP-SLAM uses iterative closest point (ICP).

Performance of AVP-SLAM-PLUS could be found in video(https://www.bilibili.com/video/BV11R4y137xb/).

<p align='center'>
<img src="images/mapping1.gif"  width = 45% height = 40% " />
<img src="images/mapping2.gif" width = 45% height = 40% />
<h5 align="center">mapping</h5>
</p>

<p align='center'>
<img src="images/localization1.gif" width = 45% height = 45% />
<img src="images/localization2.gif" width = 45% height = 45% />
<h5 align="center">localization</h5>
</p>
                  
The AVP-SLAM-PLUS code is simple and developed to be a good demonstrative example for SLAM beginners. The framework of the original structure of AVP-SLAM-PLUS is as follows:

<p align='center'>
<img src="images/avp_slam_plus_frame.PNG" width = 55% height = 55% />
<h5 align="center">AVP-SLAM-PLUS Framework</h5>
</p>

During initial testing, AVP-SLAM-PLUS produced a trajectory with inconsistent scaling and frames between Gazebo and RViz. Additionally, the performance of SLAM using the multi-RGBD mode was failure prone. While AVP-SLAM-PLUS consistently found a solution when run in multi-RGB mode, the trajectory resulting from SLAM tracked well initially, but was "scaled down" as time went on and was also not smooth overall. AVP-SLAM++ works to solve these problems with the following steps.
- Implementing an odometry controller to produce simulated transformations between each subsequent pose
- Extracting multi-RGB mode AVP-SLAM-PLUS poses
- Optimizing the resulting trajectory using GraphSLAM, both using a batch solution and also ISAM2. 
  - AVP-SLAM-PLUS poses were used as verticies
  - Odometry transformations were used as edges
  - Loop closures were found using the distance between two AVP-SLAM-PLUS poses. These poses are already localized using either ICP or NDT, and thus, loop closure is found if two poses are within a threshold distance of each other. In order to not produce loop closure constraints between neighboring (or truly close points), 300 neighboring poses were ignored for this comparison. 300 was found using tuning
## Folder Structure
    .
    ├── GraphSLAM                         # Python script to parse rosbag and generate graph
    ├── avp_slam_plus                     # source code
    │   ├── config
    │   ├── data
    │   ├── include
    │   ├── launch
    │   ├── model
    │   ├── scripts
    │   └── src
    ├── controller                        # Script to automate trajectory generation in Gazebo
    ├── convert_orientation               # 
    ├── data_analysis                     # 
    ├── images
    ├── parse_rosbag
    ├── simulate_gazebo
    └── README.md
## 1. Prerequisites
### 1.1 Operating System Basics
Ubuntu 64-bit. The version of your kernel (18.04, 20.04, etc.) does not matter as long as it supports `docker`.

### 1.2 Clone Repository and Docker Setup
Running this environment locally on an Ubuntu system may lead to issues. We have provided a [docker image and shell script](BROKEN) for convenience.

In order to proceed with setup, you must have docker installed on your local system. For Ubuntu 20.04, follow Step 1 and 2 [here](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-3-using-the-docker-command).

Once you have downloaded the docker image, navigate to the directory where this is stored and load the image onto your system.
```
    cd ~/path/to/docker/image
    docker load --input avp-slam.tar
```

Clone the AVP-SLAM++ repository to you local system. This does not need to be in the same location as `avp-slam.tar`, as this repository will be used often
```
    git clone https://github.com/rob530-w22-team25/AVP-SLAM-PLUS.git
```

Edit the shell script to utilize the path to your AVP-SLAM repository, save it, and run your docker image with the following command. By linking your local AVP-SLAM repository to your docker image, you will be able to make changes locally and also run the most up-to-date code in the docker image.
```
    cd ~/path/to/shell/script
    ./avp-slam.sh
```

Whenever you have updated your docker environment, and would like to save, use the `docker commit command`. While the edited docker image is running, execute
```
    docker ps
```
and copy the value of the `NAMES` field of the image that you are currently running. Then execute the `commit` command with the following:
```
    docker commit NAME_FROM_PS_COMMAND avp-slam
```
You will receive a `SHA256` line as output to confirm the completion of the command. This will now allow you to utilize the latest version of your docker image the next time you run it with the provided shell script. Failure to do this after you have made changes will force you to enact your changes all over again. If you have gotten to this point in the docker setup, commit your progress to your docker image at least once to ensure models too do not need to be loaded again.

### 1.3 **Load Gazebo Model** 
Inside your docker image, run the following commands to load the models necessary for use in Gazebo.

```
    cd /home/catkin_ws/AVP-SLAM-PLUS/avp_slam_plus/model/
    unzip my_ground_plane.zip -d ~/.gazebo/models/
```
Depending on how you have configured your docker paths, the first path may be slightly different. Regardless, navigate to the `models/` folder inside of AVP-SLAM-PLUS and then run the `unzip` command. 

## 2. Build AVP-SLAM-PLUS
```
    cd /home/catkin_ws
    catkin_make
    source /home/catkin_ws/devel/setup.bash
```

## 3. RUN Example
### 3.1  **RGB Mode**
                  
#### **save map**

if you want to save map and use the map to do localization, you should ensure your config file have be correctely set. The config file is at   **AVP-SLAM-PLUS/avp_slam_plus/configFile.yaml**

```
    mapSave: true
    mapSaveLocation: your map file address 
```                 
                  
#### 3.1.1  **Mapping**
```
    roslaunch avp_slam_plus slamRGB.launch
```

open a new terminal, control robot move. 
```
    roslaunch robot_control robot_control.launch
```
if you firstly control robot move, you should ensure **robot_control.py** in **AVP-SLAM-PLUS/simlate_gazebo/robot_control/** to be executable. you can do this command to let **robot_control.py** to be executable.
```
    chmod +777 robot_control.py
```                 

#### 3.1.2  **Localization**
if you have do 3.1.1 and "save map", you can do localization in the prior map.
```
    roslaunch avp_slam_plus localizationRGB.launch
```


open a new terminal, control robot move
```
    roslaunch robot_control robot_control.launch
```

### 3.2  **RGBD Mode**
 
#### **save map**

if you want to save map and use the map to do localization, you should ensure your config file have be correctely set. The config file is at   **AVP-SLAM-PLUS-main/avp_slam_plus/configFile.yaml**
```
    mapSave: true
    mapSaveLocation: your map file address 
```   
                               
                               
#### 3.2.1  **Mapping**
```
    roslaunch avp_slam_plus slamRGBD.launch
```

open a new terminal, control robot move
```
    roslaunch robot_control robot_control.launch
```


#### 3.2.2  **Localization**
if you have do 3.2.1 and "save map", you can do localization in the prior map.

```
    roslaunch avp_slam_plus localizationRGBD.launch
```

open a new terminal, control robot move
```
    roslaunch robot_control robot_control.launch
```

## 4.Acknowledgements
We'd like to thank the original AVP-SLAM team, Tong Qin, Tongqing Chen, Yilun Chen, and Qing Su. Additionally, we would also like to acknowledge the precusory work done by [TurtleZhong](https://github.com/TurtleZhong/AVP-SLAM-SIM) who first developed an initial simulation environment for AVP-SLAM and by [huchunxu](https://github.com/huchunxu/ros_exploring) who developed an intutive simulated robot model. Addtionally, a big thanks to [Liu Guitao](mailto:liuguitao@sia.cn) who originally developed AVP-SLAM-PLUS. The original implementation of AVP-SLAM-PLUS can be found [here](https://github.com/liuguitao/AVP-SLAM-PLUS).

Additionally, we would like to acknowledge and give a big thanks to the W22 instructional team of [NAVARCH 568/ROB 530 Mobile Robotics](https://robots.engin.umich.edu/mobilerobotics/) for their teaching and continual support throughout this entire process. We appreciated the effort and the learning opportunity.
