# README

中文 | [English](README.md) 

1. 项目名称：ros-noetic-jazzy-noble
2. 版本：v1.0
3. 版本状态：预发行版本
4. 作者：Tung Chia-hui
5. 网站：[https://github.com/tungchiahui](https://github.com/tungchiahui)
6. 电子邮件：tungchiahui@gmail.com
7. 团队：山东理工大学机电创新学会Vinci机器人队
8. 构建日期：2024-10-03
9. 未来功能：
    1. 查找错误。


# 项目说明

## 1. `拉取仓库`

### 功能

- 仓库链接：[https://hub.docker.com/r/tungchiahui/ros-noetic-jazzy-noble]https://hub.docker.com/r/tungchiahui/ros-noetic-jazzy-noble

### 使用方法

1. **拉取镜像**

   使用以下命令编译出二进制文件：

   ```bash
    # 拉取仓库镜像
    docker pull tungchiahui/ros-noetic-jazzy-noble


2. **创建容器**

   使用以下命令创建：

   ```bash
    # 如果是X11请运行下方命令，请记得改电脑用户名yearner
    sudo docker run  -dit \
    --gpus all \
    -e NVIDIA_DRIVER_CAPABILITIES=all \
    --name=ros_noetic_jazzy_noble \     #容器名称，随便起
    -v /home/yearner:/home/yearner \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v /dev/dri:/dev/dri \
    --device=/dev/snd \
    --device=/dev/dri/renderD128 \
    -e DISPLAY=unix$DISPLAY \
    -w /home/yearner tungchiahui/ros-noetic-jazzy-noble:latest

    # 如果是Wayland请运行下方命令，请记得改电脑用户名yearner
    sudo docker run -dit \
    --gpus all \
    -e NVIDIA_DRIVER_CAPABILITIES=all \
    --name=ros_noetic_jazzy_noble \
    -v /home/yearner:/home/yearner \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v /dev/dri:/dev/dri \
    --device=/dev/snd \
    --device=/dev/dri/renderD128 \
    -e DISPLAY=:0 \
    -w /home/yearner tungchiahui/ros-noetic-jazzy-noble:latest
