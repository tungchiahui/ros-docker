# 使用Ubuntu 20.04 AMD64基础镜像
FROM ubuntu:20.04

# 设置非交互式安装以避免tzdata提示
ENV DEBIAN_FRONTEND=noninteractive

# 强制使用 Bash 作为默认 Shell
SHELL ["/bin/bash", "-c"]

# 先更新包列表并安装CA证书
RUN apt-get update && apt-get install -y ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# 替换为北京外国语大学Ubuntu镜像源（AMD64专用）
RUN sed -i 's|http://archive.ubuntu.com/ubuntu|https://mirrors.bfsu.edu.cn/ubuntu|g' /etc/apt/sources.list

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
    libpcl-dev \
    libeigen3-dev \
    libboost-dev \
    && rm -rf /var/lib/apt/lists/*

# 配置终端颜色和提示符
RUN sed -i 's/#force_color_prompt=yes/force_color_prompt=yes/g' /root/.bashrc && \
    # 自定义PS1（带颜色的提示符）
    echo 'export PS1="\[\033[1;36m\]\u@\h\[\033[00m\]:\[\033[1;34m\]\w\[\033[00m\]\$ "' >> /root/.bashrc && \
    # 给ls、grep等命令添加别名
    echo "alias ls='ls --color=auto'" >> /root/.bashrc && \
    echo "alias grep='grep --color=auto'" >> /root/.bashrc && \
    echo "alias ll='ls -alFh --color=auto'" >> /root/.bashrc && \
    # 清理临时文件
    rm -rf /tmp/LS_COLORS


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
    ros-noetic-moveit \
    ros-noetic-serial \
    python3-rosdep \
    python3-rosinstall \
    python3-rosinstall-generator \
    python3-wstool \
    && rm -rf /var/lib/apt/lists/*

# 启用NVIDIA显卡支持（仅当使用NVIDIA显卡时）
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=all

# 设置QT的显示平台为xcb
# ENV QT_QPA_PLATFORM=xcb

# 配置rosdep使用镜像源
RUN echo "# 配置ROS1环境" >> /root/.bashrc
RUN mkdir -p /etc/ros/rosdep/sources.list.d/
RUN curl -o /etc/ros/rosdep/sources.list.d/20-default.list -L https://mirrors.bfsu.edu.cn/github-raw/ros/rosdistro/master/rosdep/sources.list.d/20-default.list
RUN echo 'export ROSDISTRO_INDEX_URL=https://mirrors.bfsu.edu.cn/rosdistro/index-v4.yaml' >> /root/.bashrc

# 设置环境变量
RUN echo "source /opt/ros/noetic/setup.bash" >> /root/.bashrc
RUN echo "export ROS_MASTER_URI=http://localhost:11311" >> /root/.bashrc
RUN echo "export ROS_HOSTNAME=localhost" >> /root/.bashrc
RUN echo "export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:/home/tungchiahui/UserFloder/MySource/ROS_WS/gazebo_models:/home/tungchiahui/UserFloder/MySource/ROS_WS/ign_models" >> /root/.bashrc

CMD ["/bin/bash"]
