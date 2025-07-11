# 支持多架构，使用变量指定架构
ARG TARGETARCH
# 使用Ubuntu 24.04作为基础镜像
FROM --platform=linux/${TARGETARCH} ubuntu:24.04

# 设置环境变量以防止交互安装
ENV DEBIAN_FRONTEND=noninteractive

# 强制使用 Bash 作为默认 Shell
SHELL ["/bin/bash", "-c"]

# 先更新包列表并安装CA证书
RUN apt-get update && apt-get install -y ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# 替换为北京外国语大学的DEB822格式镜像源
RUN if [ "$(dpkg --print-architecture)" = "amd64" ]; then \
        sed -i 's|http://archive.ubuntu.com/ubuntu/|https://mirrors.bfsu.edu.cn/ubuntu/|g' /etc/apt/sources.list.d/ubuntu.sources; \
    elif [ "$(dpkg --print-architecture)" = "arm64" ]; then \
        sed -i 's|http://ports.ubuntu.com/ubuntu-ports/|https://mirrors.bfsu.edu.cn/ubuntu-ports/|g' /etc/apt/sources.list.d/ubuntu.sources; \
    fi

# 更新包管理器，安装必要工具（locales, software-properties-common, curl, gnupg2）
RUN apt-get update && apt-get install -y \
    locales \
    software-properties-common \
    curl \
    gnupg2 \
    lsb-release \
    usbutils \
    udev \
    hwinfo \
    libgl1-mesa-dev \
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
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] https://mirrors.bfsu.edu.cn/ros2/ubuntu noble main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null

# 更新包管理器并安装ROS 2 Jazzy desktop
RUN apt-get update && apt-get install -y \
    ros-jazzy-desktop \
    ros-dev-tools \
    ros-jazzy-xacro \
    ros-jazzy-joint-state-publisher \
    ros-jazzy-joint-state-publisher-gui \
    ros-jazzy-ros-gz \
    ros-jazzy-diagnostic-updater \
    ros-jazzy-navigation2 \
    ros-jazzy-nav2-bringup \
    ros-jazzy-slam-toolbox \
    ros-jazzy-cartographer \
    ros-jazzy-cartographer-ros \
    ros-jazzy-asio-cmake-module \
    ros-jazzy-serial-driver \
    ros-jazzy-pcl-ros \
    ros-jazzy-vision-opencv \
    ros-jazzy-pointcloud-to-laserscan \
    libpcl-dev \
    libeigen3-dev \
    libpcap-dev \
    python3-colcon-common-extensions \
    && rm -rf /var/lib/apt/lists/*


# 启用NVIDIA显卡支持（仅当使用NVIDIA显卡时）
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=all

# 设置QT的显示平台为xcb
# ENV QT_QPA_PLATFORM=xcb

# 配置rosdep使用镜像源
RUN echo "# 配置ROS2环境" >> /root/.bashrc
RUN mkdir -p /etc/ros/rosdep/sources.list.d/
RUN curl -o /etc/ros/rosdep/sources.list.d/20-default.list -L https://mirrors.bfsu.edu.cn/github-raw/ros/rosdistro/master/rosdep/sources.list.d/20-default.list
RUN echo 'export ROSDISTRO_INDEX_URL=https://mirrors.bfsu.edu.cn/rosdistro/index-v4.yaml' >> /root/.bashrc

# 设置环境变量
RUN echo "source /opt/ros/jazzy/setup.bash" >> /root/.bashrc
RUN echo "export ROS_DOMAIN_ID=6" >> /root/.bashrc
RUN echo "export GZ_SIM_RESOURCE_PATH=$GZ_SIM_RESOURCE_PATH:/home/tungchiahui/UserFloder/MySource/ROS_WS/gazebo_models:/home/tungchiahui/UserFloder/MySource/ROS_WS/ign_models" >> /root/.bashrc

# 配置clangd必备CMake选项
RUN mkdir -p /root/.colcon && \
    echo -e "build:\n  cmake-args:\n    - -DCMAKE_EXPORT_COMPILE_COMMANDS=ON" > /root/.colcon/defaults.yaml

# 启动时默认进入bash shell
CMD ["bash"]
