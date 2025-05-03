# 支持多架构，使用变量指定架构
ARG TARGETARCH
# 基于NVIDIA官方CUDA 12.6和CuDNN基础镜像
FROM --platform=linux/${TARGETARCH} nvidia/cuda:12.8.0-cudnn-devel-ubuntu22.04

# 设置环境变量以防止交互安装
ENV DEBIAN_FRONTEND=noninteractive

# 强制使用 Bash 作为默认 Shell
SHELL ["/bin/bash", "-c"]

# 先更新包列表并安装CA证书
RUN apt-get update && apt-get install -y ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# 替换为北京外国语大学的镜像源
RUN if [ "$(dpkg --print-architecture)" = "amd64" ]; then \
        sed -i 's|http://archive.ubuntu.com/ubuntu/|https://mirrors.bfsu.edu.cn/ubuntu/|g' /etc/apt/sources.list; \
    elif [ "$(dpkg --print-architecture)" = "arm64" ]; then \
        sed -i 's|http://ports.ubuntu.com/ubuntu-ports/|https://mirrors.bfsu.edu.cn/ubuntu-ports/|g' /etc/apt/sources.list; \
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
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] https://mirrors.bfsu.edu.cn/ros2/ubuntu jammy main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null

# 更新包管理器并安装ROS 2 Humble desktop
RUN apt-get update && apt-get install -y \
    ros-humble-desktop \
    ros-dev-tools \
    ros-humble-xacro \
    ros-humble-joint-state-publisher \
    ros-humble-joint-state-publisher-gui \
    ros-humble-ros-gz \
    ros-humble-diagnostic-updater \
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
    && rm -rf /var/lib/apt/lists/*


# 配置rosdep使用镜像源
RUN echo "# 配置ROS2环境" >> /root/.bashrc
RUN mkdir -p /etc/ros/rosdep/sources.list.d/
RUN curl -o /etc/ros/rosdep/sources.list.d/20-default.list -L https://mirrors.bfsu.edu.cn/github-raw/ros/rosdistro/master/rosdep/sources.list.d/20-default.list
RUN echo 'export ROSDISTRO_INDEX_URL=https://mirrors.bfsu.edu.cn/rosdistro/index-v4.yaml' >> /root/.bashrc

# 设置环境变量
RUN echo "source /opt/ros/humble/setup.bash" >> /root/.bashrc
RUN echo "export ROS_DOMAIN_ID=6" >> /root/.bashrc
RUN echo "export IGN_GAZEBO_RESOURCE_PATH=$IGN_GAZEBO_RESOURCE_PATH:/home/tungchiahui/UserFloder/MySource/ROS_WS/gazebo_models:/home/tungchiahui/UserFloder/MySource/ROS_WS/ign_models" >> /root/.bashrc

# 配置clangd必备CMake选项
RUN mkdir -p /root/.colcon && \
    echo -e "build:\n  cmake-args:\n    - -DCMAKE_EXPORT_COMPILE_COMMANDS=ON" > /root/.colcon/defaults.yaml


# 安装必要工具
RUN apt-get update && apt-get install -y \
    libcurl4 build-essential pkg-config cmake-gui unzip\
    libopenblas-dev libeigen3-dev libtbb-dev \
    libavcodec-dev libavformat-dev \
    libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev \
    libswscale-dev libgtk-3-dev libpng-dev libjpeg-dev \
    libcanberra-gtk-module libcanberra-gtk3-module libv4l-dev python3.10-dev python3-numpy \
    && rm -rf /var/lib/apt/lists/*

# 克隆opencv源码
WORKDIR /opt
RUN wget https://ghfast.top/https://github.com/opencv/opencv/archive/refs/tags/4.11.0.zip -O opencv-4.11.0.zip && \
    wget https://ghfast.top/https://github.com/opencv/opencv_contrib/archive/refs/tags/4.11.0.zip -O opencv_contrib-4.11.0.zip && \
    unzip opencv-4.11.0.zip && \
    unzip opencv_contrib-4.11.0.zip

# CMake编译opencv-cuda
RUN mkdir /opt/opencv-4.11.0/build && cd /opt/opencv-4.11.0/build && \
    rm -rf ./* && \
    if [ "$(dpkg --print-architecture)" = "amd64" ]; then \
        cmake .. -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/usr/local \
        -DBUILD_SHARED_LIBS=ON \
        -DOPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.11.0/modules \
        -DOPENCV_ENABLE_NONFREE=ON \
        -DBUILD_TESTS=OFF \
        -DBUILD_PERF_TESTS=OFF \
        -DOPENCV_GENERATE_PKGCONFIG=ON \
        -DWITH_GTK=ON \
        -DWITH_CUDA=ON \
        -DENABLE_FAST_MATH=ON \
        -DCUDA_FAST_MATH=ON \
        -DWITH_CUBLAS=ON \
        -DCUDA_ARCH_BIN="6.1;7.5;8.6;8.9;12.0" \
        -DCUDA_ARCH_PTX="12.0" \
        -DCUDA_HOST_COMPILER=/usr/bin/gcc-11 \
        -DWITH_CUDNN=ON \
        -DOPENCV_DNN_CUDA=ON \
        -DWITH_IPP=ON \
        -DWITH_TBB=ON \
        -DWITH_OPENMP=ON \
        -DWITH_PTHREADS_PF=ON \
        -DOPENCV_PYTHON3_VERSION=3.10 \
        -DPYTHON3_EXECUTABLE=/usr/bin/python3 \
        -DPYTHON3_LIBRARY=$(python3 -c "import sysconfig; from os.path import join; print(join(sysconfig.get_config_var('LIBDIR'), sysconfig.get_config_var('LDLIBRARY')))") \
        -DPYTHON3_NUMPY_INCLUDE_DIRS=$(python3 -c "import numpy; print(numpy.get_include())") \
        -DPYTHON3_PACKAGES_PATH=$(python3 -c "import sysconfig; print(sysconfig.get_path('purelib'))") \
        -DPYTHON3_INCLUDE_DIR=$(python3 -c "import sysconfig; print(sysconfig.get_path('include'))") \
        -DWITH_OPENGL=ON; \
    elif [ "$(dpkg --print-architecture)" = "arm64" ]; then \
        cmake .. -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/usr/local \
        -DBUILD_SHARED_LIBS=ON \
        -DOPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.11.0/modules \
        -DOPENCV_ENABLE_NONFREE=ON \
        -DBUILD_TESTS=OFF \
        -DBUILD_PERF_TESTS=OFF \
        -DOPENCV_GENERATE_PKGCONFIG=ON \
        -DWITH_GTK=ON \
        -DWITH_CUDA=ON \
        -DENABLE_FAST_MATH=ON \
        -DCUDA_FAST_MATH=ON \
        -DWITH_CUBLAS=ON \
        -DCUDA_ARCH_BIN="7.2;8.7" \
        -DCUDA_ARCH_PTX="8.7" \
        -DCUDA_HOST_COMPILER=/usr/bin/gcc-11 \
        -DWITH_CUDNN=ON \
        -DOPENCV_DNN_CUDA=ON \
        -DWITH_IPP=OFF \
        -DWITH_TBB=ON \
        -DWITH_OPENMP=ON \
        -DWITH_PTHREADS_PF=ON \
        -DOPENCV_PYTHON3_VERSION=3.10 \
        -DPYTHON3_EXECUTABLE=/usr/bin/python3 \
        -DPYTHON3_LIBRARY=$(python3 -c "import sysconfig; from os.path import join; print(join(sysconfig.get_config_var('LIBDIR'), sysconfig.get_config_var('LDLIBRARY')))") \
        -DPYTHON3_NUMPY_INCLUDE_DIRS=$(python3 -c "import numpy; print(numpy.get_include())") \
        -DPYTHON3_PACKAGES_PATH=$(python3 -c "import sysconfig; print(sysconfig.get_path('purelib'))") \
        -DPYTHON3_INCLUDE_DIR=$(python3 -c "import sysconfig; print(sysconfig.get_path('include'))") \
        -DWITH_OPENGL=ON; \
    fi

# 编译opencv_cuda
RUN cd /opt/opencv-4.11.0/build && \
    make all -j$(grep -c ^processor /proc/cpuinfo) && \
    make install -j$(grep -c ^processor /proc/cpuinfo) && \
    ldconfig

RUN rm -rf /opt/opencv*

# 设置环境变量
RUN echo "# 设置 LD_LIBRARY_PATH" >> /root/.bashrc
RUN echo 'export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"' >> /root/.bashrc

RUN apt-get update && apt-get install -y \
    python3-numpy \
    libboost-python-dev \
    ros-humble-rcpputils \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt
RUN wget https://ghfast.top/https://github.com/tungchiahui/cv_bridge_fix_link/archive/refs/tags/humble.zip -O cv_bridge.zip && \
    unzip cv_bridge.zip && \
    cd ./cv_bridge_fix_link-humble && \
    source /opt/ros/humble/setup.bash && \
    rosdep update && \
    colcon build --symlink-install --allow-overriding cv_bridge image_geometry

RUN echo '# 配置ROS2 CV_Bridge' >> /root/.bashrc
RUN echo 'source /opt/cv_bridge_fix_link-humble/install/setup.bash' >> /root/.bashrc

# 启动时默认进入bash shell
CMD ["bash"]
