# Base image
FROM docker.io/arm64v8/ubuntu:22.04

# Environment variables
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES all
ARG UID=1000
ARG GID=1000
ENV USERNAME jetson
ENV HOME /home/$USERNAME

# Create non-root user with sudo privileges
RUN useradd -m $USERNAME && \
    echo "$USERNAME:$USERNAME" | chpasswd && \
    usermod --shell /bin/bash $USERNAME && \
    usermod -aG sudo $USERNAME && \
    mkdir /etc/sudoers.d && \
    echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/$USERNAME && \
    chmod 0440 /etc/sudoers.d/$USERNAME && \
    usermod  --uid $UID $USERNAME && \
    groupmod --gid $GID $USERNAME && \
    gpasswd -a $USERNAME video

# Set non-interactive environment
ARG DEBIAN_FRONTEND=noninteractive

# Install essential packages
RUN apt-get update && \
    apt-get install -qq -y --no-install-recommends \
    # Basic utilities
    bash-completion \
    bc \
    build-essential \
    bzip2 \
    can-utils \
    ca-certificates \
    cmake \
    command-not-found \
    curl \
    emacs \
    freeglut3-dev \
    git \
    gnupg2 \
    # Multimedia libraries
    gstreamer1.0-alsa \
    gstreamer1.0-libav \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-plugins-base \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-ugly \
    gstreamer1.0-tools \
    # Development tools
    i2c-tools \
    iproute2 \
    iputils-ping \
    iw \
    kbd \
    kmod \
    language-pack-en-base \
    libapt-pkg-dev \
    libcanberra-gtk3-module \
    libgles2 \
    libglu1-mesa-dev \
    libglvnd-dev \
    libgtk-3-0 \
    libudev1 \
    libvulkan1 \
    libzmq5 \
    mesa-utils \
    mtd-utils \
    parted \
    pciutils \
    python3 \
    python3-distutils \
    python3-numpy \
    python3-pexpect \
    python3-pip \
    sox \
    sudo \
    tmux \
    udev \
    vulkan-tools \
    wget \
    wireless-tools \
    wpasupplicant \
    && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# EGL configurations
RUN echo "/usr/lib/aarch64-linux-gnu/tegra" >> /etc/ld.so.conf.d/nvidia-tegra.conf && \
    echo "/usr/lib/aarch64-linux-gnu/tegra-egl" >> /etc/ld.so.conf.d/nvidia-tegra.conf && \
    rm -rf /usr/share/glvnd/egl_vendor.d && \
    mkdir -p /usr/share/glvnd/egl_vendor.d/ && \
    echo '{"file_format_version":"1.0.0","ICD":{"library_path":"libEGL_nvidia.so.0"}}' > /usr/share/glvnd/egl_vendor.d/10_nvidia.json && \
    mkdir -p /usr/share/egl/egl_external_platform.d/ && \
    echo '{"file_format_version":"1.0.0","ICD":{"library_path":"libnvidia-egl-wayland.so.1"}}' > /usr/share/egl/egl_external_platform.d/nvidia_wayland.json

# NVIDIA apt repository
RUN echo "deb https://repo.download.nvidia.com/jetson/common r35.3 main" >> /etc/apt/sources.list.d/nvidia-l4t-apt-source.list && \
    echo "deb https://repo.download.nvidia.com/jetson/t234 r35.3 main" >> /etc/apt/sources.list.d/nvidia-l4t-apt-source.list && \
    wget -O /etc/jetson-ota-public.key https://gitlab.com/nvidia/container-images/l4t-base/-/raw/master/jetson-ota-public.key && \
    apt-key add /etc/jetson-ota-public.key

# CUDA, cuDNN, TensorRT
RUN apt-get update && \
    apt-get install -qq -y --no-install-recommends \
    cuda \
    libcudnn8 \
    libcudnn8-dev \
    libcudnn8-samples \
    tensorrt \
    tensorrt-libs \
    tensorrt-dev \
    libnvinfer8 \
    libnvinfer-dev \
    gcc-9 \
    g++-9 \
    && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    ln -s /usr/bin/gcc-9 /usr/local/cuda/bin/gcc && \
    ln -s /usr/bin/g++-9 /usr/local/cuda/bin/g++ && \
    ldconfig

# ROS2 installation
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    ros-humble-desktop \
    ros-dev-tools \
    && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Colcon and rosdep setup
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python3-colcon-common-extensions \
    python3-rosdep \
    && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    sudo rosdep init && \
    rosdep update

# Set user environment
USER $USERNAME
WORKDIR /home/$USERNAME
SHELL ["/bin/bash", "-l", "-c"]

# Add CUDA paths to bashrc
RUN echo "export PATH=/usr/local/cuda/bin:$PATH" >> ~/.bashrc && \
    echo "export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH" >> ~/.bashrc && \
    echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc && \
    echo "source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash" >> ~/.bashrc
