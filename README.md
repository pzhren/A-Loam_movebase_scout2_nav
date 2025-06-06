# A-Loam_movebase_scout2_nav
A-loam纯Robosense SLAM建图，movebase导航，scout2松灵小车导航
# 环境
要求：ubuntu20.04 ROS1 noetic  ceres 1.14，robosense32 线
## Step1
```
    cd ~/catkin_ws/src
    git clone https://github.com/HKUST-Aerial-Robotics/A-LOAM.git
    cd ../
    catkin_make
```
## Step2
```
sudo gedit /etc/apt/sources.list
deb http://cz.archive.ubuntu.com/ubuntu trusty main universe
```
 
```
sudo apt-get install liblapack-dev libsuitesparse-dev libcxsparse3.1.2 libgflags-dev
sudo apt-get install libgoogle-glog-dev libgtest-dev
```
## Step3
```
wget ceres-solver.org/ceres-solver-1.14.0.tar.gz
tar -zxvf ceres-solver-1.14.0.tar.gz
```
## Step4
```
cd ceres-solver-1.14.0 
mkdir build
cd build
cmake .. 
make -j4
sudo make install
```
# 修改源码：
```
//改A-LOAM下的CMakeLists.txt
 
c++11 改为 c++14
 
 
//改A-LOAM/src/kittiHelper.cpp
 
CV_LOAD_IMAGE_GRAYCALE 改为 cv::IMREAD_GRAYSCALE       //91行和93行
 
 
//改A-LOAM/src/scanRegistration.cpp里的
 
#include <opencv/cv.h> 改为 #include <opencv2/imgproc.hpp>
 
 
//改A-LOAM/src/三个cpp文件laserMapping.cpp和scanRegistration.cpp和laserOdometry.cpp
 
//使用Ctrl+H快捷键快速替换
 
/camera_init 改为 camera_init

```
![image](https://github.com/user-attachments/assets/f4731bcb-3e86-4160-b58e-2fb0a7f2e5ef)

```
//将scanRegistration.cpp中的sub接口改一下
 
ros::Subscriber subLaserCloud = nh.subscribe<sensor_msgs::PointCloud2>("/velodyne_points", 100, laserCloudHandler);
 
//改为
 
ros::Subscriber subLaserCloud = nh.subscribe<sensor_msgs::PointCloud2>("/points_raw", 100, laserCloudHandler);
 
//改完后保存
```

![image](https://github.com/user-attachments/assets/f65b4a8f-e2ba-4a7d-930d-1a73fd9fb24d)


Scout2 导航系统方案(amcl 定位+map server+movebase导航)，导航部分

##编写小车启动launch文件，包括以下几个部分：

1 小车模型：下载scout_ros自带的scout_v2.urdf,  改写4轮坐标系到小车本体base_link的转换
launch文件在scout_ros/bringup/nav4，主要更改里面move_base节点，4个yaml文件参数
 vscode 打开catkin_zk

2 激光雷达：下载速腾激光雷达驱动，对雷达参数，包括雷达类型和坐标系改写，打开激光雷达
```
roslaunch rslidar_sdk start.launch
```

3 点云转换激光雷达扫描数据：下载pointcloud_to_laserscan包，获取速腾rslidar_points节点到laser转        
换。
```
git clone https://github.com/ros-perception/pointcloud_to_laserscan/tree/lunar-devel
```
 更改launch文件，sample_node.launch里面 <remap from="cloud_in" to="$(arg camera)/depth_registered/points_processed"/>
 为 <remap from="cloud_in" to="/rslidar_points"/>
4 打开地图服务：将图进行节点发布，并编写图的中心点，占用率等参数
```
rosrun mapserver mapserver map.yaml
```

5 编写map→odom→baselink 的tf转换树，进行坐标系转换
参考nav4内容,自行修改

6 下载AMCL 自适应蒙特卡洛粒子波定位包，初始化小车位置，里程计坐标，小车坐标和地图坐标，运行AMCL定位节点
AMCL节点，主要修改config.yaml文件

 7 下载movebase导航包，编写局部teb动态窗口局部规划参数，navfn全局规划参数，常规代价地图，全局代价地图，局部代价地图参数文件,启动导航
 在rviz打开map，订阅节点map,打开robotmodel,打开path,观察仿真环境小车运动。
