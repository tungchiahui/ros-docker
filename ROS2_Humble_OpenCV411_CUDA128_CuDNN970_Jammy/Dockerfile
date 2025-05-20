# 支持多架构，使用变量指定架构
ARG TARGETARCH
# 基于NVIDIA官方CUDA 12.8和CuDNN基础镜像
FROM --platform=linux/${TARGETARCH} tungchiahui/opencv:411-cuda128-cudnn971-jammy

# 生成和设置en_US.UTF-8 locale
RUN locale-gen en_US en_US.UTF-8 && \
    update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8

# 设置环境变量以确保locale被正确使用
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8


# 添加universe存储库
RUN add-apt-repository universe

# 添加ROS 2的密钥和源
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] https://mirrors.bfsu.edu.cn/ros2/ubuntu jammy main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null

# 更新包管理器并安装ROS 2 Humble desktop
RUN if [ "$TARGETARCH" = "amd64" ]; then \
        apt-get update && apt-get install -y \
        ros-humble-desktop \
        ros-dev-tools \
        ros-humble-xacro \
        ros-humble-joint-state-publisher \
        ros-humble-joint-state-publisher-gui \
        ros-humble-ros-gz \
        ros-humble-gazebo-ros \
        ros-humble-gazebo-ros-pkgs \
        ros-humble-diagnostic-updater \
        ros-humble-teleop-twist-keyboard \
        ros-humble-navigation2 \
        ros-humble-nav2-bringup \
        ros-humble-slam-toolbox \
        ros-humble-cartographer \
        ros-humble-cartographer-ros \
        ros-humble-asio-cmake-module \
        ros-humble-serial-driver \
        ros-humble-pcl-ros \
        ros-humble-vision-opencv \
        ros-humble-pointcloud-to-laserscan \
        libpcl-dev \
        libeigen3-dev \
        libpcap-dev \
        python3-colcon-common-extensions \
        && rm -rf /var/lib/apt/lists/*; \
    elif [ "$TARGETARCH" = "arm64" ]; then \
        apt-get update && apt-get install -y \
        ros-humble-desktop \
        ros-dev-tools \
        ros-humble-xacro \
        ros-humble-joint-state-publisher \
        ros-humble-joint-state-publisher-gui \
        ros-humble-ros-gz \
        ros-humble-diagnostic-updater \
        ros-humble-teleop-twist-keyboard \
        ros-humble-navigation2 \
        ros-humble-nav2-bringup \
        ros-humble-slam-toolbox \
        ros-humble-cartographer \
        ros-humble-cartographer-ros \
        ros-humble-asio-cmake-module \
        ros-humble-serial-driver \
        ros-humble-pcl-ros \
        ros-humble-vision-opencv \
        ros-humble-pointcloud-to-laserscan \
        libpcl-dev \
        libeigen3-dev \
        libpcap-dev \
        python3-colcon-common-extensions \
        && rm -rf /var/lib/apt/lists/*; \
    fi


# 配置rosdep使用镜像源
RUN echo "# 配置ROS2环境" >> /root/.bashrc
RUN mkdir -p /etc/ros/rosdep/sources.list.d/
RUN curl -o /etc/ros/rosdep/sources.list.d/20-default.list -L https://mirrors.bfsu.edu.cn/github-raw/ros/rosdistro/master/rosdep/sources.list.d/20-default.list
RUN echo 'export ROSDISTRO_INDEX_URL=https://mirrors.bfsu.edu.cn/rosdistro/index-v4.yaml' >> /root/.bashrc

# 设置环境变量
RUN echo "source /opt/ros/humble/setup.bash" >> /root/.bashrc
RUN echo "export ROS_DOMAIN_ID=6" >> /root/.bashrc
RUN echo "# Ingition Gazebo资源配置" >> /root/.bashrc
RUN echo "export IGN_GAZEBO_RESOURCE_PATH=$IGN_GAZEBO_RESOURCE_PATH:/home/tungchiahui/UserFolder/MySource/ROS_WS/gazebo_models:/home/tungchiahui/UserFolder/MySource/ROS_WS/ign_models" >> /root/.bashrc
RUN echo "# Gazebo Classic资源配置" >> /root/.bashrc
RUN echo "export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:/home/tungchiahui/UserFolder/MySource/ROS_WS/gazebo_models:/home/tungchiahui/UserFolder/MySource/ROS_WS/ign_models" >> /root/.bashrc

# 配置clangd必备CMake选项
RUN mkdir -p /root/.colcon && \
    echo -e "build:\n  cmake-args:\n    - -DCMAKE_EXPORT_COMPILE_COMMANDS=ON" > /root/.colcon/defaults.yaml


# 编译cv_bridge(在arm64上编译报错暂时没找到解决方案)   
RUN apt-get update && apt-get install -y \
    python3-numpy \
    libboost-python-dev \
    ros-humble-rcpputils \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt
RUN if [ "$TARGETARCH" = "amd64" ]; then \
        wget --no-check-certificate https://ghfast.top/https://github.com/tungchiahui/cv_bridge_fix_link/archive/refs/tags/humble.zip -O cv_bridge.zip && \
        unzip cv_bridge.zip && \
        cd ./cv_bridge_fix_link-humble && \
        source /opt/ros/humble/setup.bash && \
        rosdep update && \
        rosdep install --from-paths src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers" && \
        colcon build --symlink-install --allow-overriding cv_bridge image_geometry; \
    fi

RUN if [ "$TARGETARCH" = "amd64" ]; then \
        echo '# 配置ROS2 CV_Bridge' >> /root/.bashrc && \
        echo 'source /opt/cv_bridge_fix_link-humble/install/setup.bash' >> /root/.bashrc; \
    fi

RUN apt-get update && apt-get install -y \
    git \
    && rm -rf /var/lib/apt/lists/*

# 安装livox-sdk2
WORKDIR /opt
RUN git clone https://github.com/Livox-SDK/Livox-SDK2.git && \
    cd ./Livox-SDK2/ && \
    mkdir build && \
    cd build && \
    cmake .. && \
    make -j$(grep -c ^processor /proc/cpuinfo) && \
    make install -j$(grep -c ^processor /proc/cpuinfo) && \
    ldconfig

# 启动时默认进入bash shell
CMD ["bash"]
