# 支持多架构，使用变量指定架构
ARG TARGETARCH
# 基于NVIDIA官方CUDA 12.8和CuDNN基础镜像
FROM --platform=linux/${TARGETARCH} tungchiahui/opencv:411-cuda128-cudnn970-focal

# 安装基础工具
RUN apt-get update && apt-get install -y \
    wget \
    gnupg2 \
    lsb-release \
    software-properties-common \
    build-essential \
    curl \
    vim \
    git \
    usbutils \
    udev \
    hwinfo \
    libgl1-mesa-dev \
    libpcl-dev \
    libeigen3-dev \
    libboost-dev \
    && rm -rf /var/lib/apt/lists/*


# 添加ROS仓库密钥
RUN sh -c 'echo "deb [arch=$(dpkg --print-architecture)] http://mirrors.bfsu.edu.cn/ros/ubuntu/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
RUN curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -

# 安装ROS Noetic桌面完整版
RUN apt-get update && apt-get install -y \
    ros-noetic-desktop-full \
    ros-noetic-rqt \
    ros-noetic-rqt-common-plugins \
    ros-noetic-rviz \
    ros-noetic-joint-state-publisher \
    ros-noetic-joint-state-publisher-gui \
    ros-noetic-xacro \
    liburdfdom-tools \
    ros-noetic-gazebo-ros-pkgs \
    gazebo11 \
    libgazebo11-dev \
    libpcap-dev \
    ros-noetic-gmapping \
    ros-noetic-map-server \
    ros-noetic-navigation \
    ros-noetic-teb-local-planner \
    ros-noetic-moveit \
    ros-noetic-serial \
    ros-noetic-pointcloud-to-laserscan \
    ros-noetic-vision-opencv  \
    python3-rosdep \
    python3-rosinstall \
    python3-rosinstall-generator \
    python3-wstool \
    && rm -rf /var/lib/apt/lists/*

# 配置rosdep使用镜像源
RUN echo "# 配置ROS1环境" >> /root/.bashrc
RUN mkdir -p /etc/ros/rosdep/sources.list.d/
RUN curl -o /etc/ros/rosdep/sources.list.d/20-default.list -L https://mirrors.bfsu.edu.cn/github-raw/ros/rosdistro/master/rosdep/sources.list.d/20-default.list
RUN echo 'export ROSDISTRO_INDEX_URL=https://mirrors.bfsu.edu.cn/rosdistro/index-v4.yaml' >> /root/.bashrc

# 设置环境变量
RUN echo "source /opt/ros/noetic/setup.bash" >> /root/.bashrc
RUN echo "export ROS_MASTER_URI=http://localhost:11311" >> /root/.bashrc
RUN echo "export ROS_HOSTNAME=localhost" >> /root/.bashrc
RUN echo "export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:/home/tungchiahui/UserFolder/MySource/ROS_WS/gazebo_models:/home/tungchiahui/UserFolder/MySource/ROS_WS/ign_models" >> /root/.bashrc

# 编译cv_bridge 
RUN apt-get update && apt-get install -y \
    python3-numpy \
    libboost-python-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt
RUN git clone -b noetic https://github.com/tungchiahui/cv_bridge_fix_link.git && \
    cd ./cv_bridge_fix_link && \
    source /opt/ros/noetic/setup.bash && \
    rosdep update && \
    rosdep install --from-paths src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers" && \
    catkin_make
    
RUN echo '# 配置ROS1 CV_Bridge' >> /root/.bashrc && \
    echo 'source /opt/cv_bridge_fix_link/install/setup.bash' >> /root/.bashrc

CMD ["/bin/bash"]
