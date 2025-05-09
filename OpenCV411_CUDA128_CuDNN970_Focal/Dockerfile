# 支持多架构，使用变量指定架构
ARG TARGETARCH
# 基于NVIDIA官方CUDA 12.8和CuDNN基础镜像
FROM --platform=linux/${TARGETARCH} nvidia/cuda:12.8.0-cudnn-devel-ubuntu20.04

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

# 安装必要工具
RUN apt-get update && apt-get install -y \
    libcurl4 build-essential pkg-config cmake-gui unzip wget\
    libopenblas-dev libeigen3-dev libtbb-dev \
    libavcodec-dev libavformat-dev \
    libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev \
    libswscale-dev libgtk-3-dev libpng-dev libjpeg-dev \
    libcanberra-gtk-module libcanberra-gtk3-module libv4l-dev python3.8-dev python3-numpy \
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
        -DCUDA_HOST_COMPILER=/usr/bin/gcc-9 \
        -DWITH_CUDNN=ON \
        -DOPENCV_DNN_CUDA=ON \
        -DWITH_IPP=ON \
        -DWITH_TBB=ON \
        -DWITH_OPENMP=ON \
        -DWITH_PTHREADS_PF=ON \
        -DOPENCV_PYTHON3_VERSION=3.8 \
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
        -DCUDA_HOST_COMPILER=/usr/bin/gcc-9 \
        -DWITH_CUDNN=ON \
        -DOPENCV_DNN_CUDA=ON \
        -DWITH_IPP=OFF \
        -DWITH_TBB=ON \
        -DWITH_OPENMP=ON \
        -DWITH_PTHREADS_PF=ON \
        -DOPENCV_PYTHON3_VERSION=3.8 \
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

# 启动时默认进入bash shell
CMD ["bash"]
