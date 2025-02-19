# openEuler 上启动小乌龟示例

## 概述

本文档主要介绍如何在 openEuler 24.03 LTS（x86 架构）环境下配置 ROS Jazzy，并成功运行最经典的 turtlesim 示例，以帮助读者快速验证 ROS 在 openEuler 上的可行性并入门使用。

## 环境

- 系统：openEuler 24.03 LTS
- 架构：x86
- ROS 版本：Jazzy

## 启动软件源

1. 需要创建一个新的 `.repo` 文件，以便添加软件源，参考以下指令：

```
cd /etc/yum.repos.d
sudo vi openEuler-jazzy.repo
```

2. `openEuler-jazzy.repo` 内容如下：

```
[openEuler-Jazzy]
name=openEuler-Jazzy
baseurl=https://eulermaker.compass-ci.openeuler.openatom.cn/api/ems1/repositories/jazzy_ament_package/openEuler%3A24.03-LTS/x86_64/
enabled=1
gpgcheck=0
```
3. 成功添加后，使用 dnf 进行更新：

```
sudo dnf clean all
sudo dnf makecache

sudo dnf update --nobest
```
## 安装 ros jazzy 

安装完整的 ros jazzy 包：
```
sudo dnf install "ros-jazzy*" --skip-broken -y
```
目前仓库中还存在 conflict 包，出现大量 conflict 报错也是预期行为， 复现时候可以忽略，dnf 会根据指令自动安装可以安装的包。

## 运行小乌龟示例

1. 安装完成之后，需要激活 ros 环境：

```
source /opt/ros/jazzy/setup.zsh
```

> 若默认使用 bash，请使用 `setup.bash`

2. 接下来确保你在图形化界面上打开两个终端窗口：

```
# terminal window1
ros2 run turtlesim turtle_teleop_key

# terminal window2
ros2 run turtlesim turtlesim_node
```
3. 选择第一个窗口开始移动，能够看到小乌龟正常移动。

预期行为：

![alt text](./img/turtlesim.png)

## 其他说明

1. **dnf 更新时出现 tbb 冲突**

在 dnf 进行更新的时候出现 tbb 冲突是预期行为。这是因为 openEuler 提供的 tbb 包不符合 ROS Jazzy 的要求，所以在仓库中存在一个版本更新的 tbb。如果出现冲突可以直接忽视。

2. **dnf 下载包时出现大量 conflict**

在 dnf 下载 ROS Jazzy 包时出现大量的 conflict 是预期行为，因为目前仓库中的包仅能够支撑启动小乌龟示例启动，所以还存在大量没有修复的 conflict 的包。

## 其他相关文档

目前正在计划编写与 ROS Jazzy 打包相关的文档：

- ROS 官方工具链调研
- 在 eulermaker 上打包全流程
- 衍生工具原理及说明 
- ...

该系列文档旨在详细的介绍 ROS 官方的打包流程，以及说明本文中的小乌龟 ROS 环境是如何构建起来的。