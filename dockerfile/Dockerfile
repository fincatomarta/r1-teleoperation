# syntax=docker/dockerfile:1
FROM elandini84/r1images:tourSim2_ubuntu_24.04_qt_6.8.3_jazzy_devel

SHELL ["/bin/bash", "-lc"]
WORKDIR /usr/local/src/robot

# Utilities/libraries useful to verify and use OpenGL through WSLg/Mesa D3D12.
# NOTE: the actual WSLg libraries are provided at runtime by mounting /usr/lib/wsl
# from the WSL host; see the docker run command below.
USER root
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        mesa-utils \
        mesa-utils-extra \
        libgl1-mesa-dri \
        mesa-vulkan-drivers && \
    rm -rf /var/lib/apt/lists/*

RUN --mount=type=secret,id=github_token,env=GITHUB_TOKEN \
    git clone https://x-access-token:${GITHUB_TOKEN}@github.com/hsp-iit/grasping-baselines.git && \
    git clone https://x-access-token:${GITHUB_TOKEN}@github.com/hsp-iit/ergocub-cartesian-control.git

WORKDIR /tmp
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh && \
    bash miniconda.sh -b -p /home/user1/miniconda3 && \
    rm miniconda.sh

ENV PATH=/home/user1/miniconda3/bin:$PATH

WORKDIR /usr/local/src/robot/ergocub-cartesian-control

RUN conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main && \
    conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r

RUN conda env create -f environment.yml

RUN source /home/user1/miniconda3/etc/profile.d/conda.sh && \
    conda activate ergocub_cartesian_control && \
    mkdir -p build && \
    cd build && \
    cmake -DCMAKE_INSTALL_PREFIX=/home/user1/robotology-superbuild/build/install .. && \
    make -j4 && \
    make install

ENV PATH=/home/user1/robotology-superbuild/build/install/bin:$PATH

# Project/runtime paths.
# /usr/lib/wsl/lib is mounted from the WSL host at docker run time and contains
# the WSL GPU/OpenGL bridge libraries needed by Mesa D3D12.
ENV LD_LIBRARY_PATH=/usr/lib/wsl/lib:/home/user1/robotology-superbuild/build/install/lib:$LD_LIBRARY_PATH
ENV YARP_DATA_DIRS=/home/user1/robotology-superbuild/build/install/share/yarp:$YARP_DATA_DIRS
ENV GZ_SIM_RESOURCE_PATH=$GZ_SIM_RESOURCE_PATH:/opt/ros/jazzy/share::/home/user1/installed/share:/home/user1/installed/share/R1Mk3/robots:/usr/local/src/robot/gz-sim-yarp-plugins/build/models:/usr/local/src/robot/LIRS-HMLG:/usr/local/src/robot/gz-sim-yarp-plugins/build/models:/usr/local/src/robot/LIRS-HMLG:/usr/local/src/robot/grasping-baselines/gazebo/models:/usr/local/src/robot/grasping-baselines/gazebo/worlds
ENV YARP_ROBOT_NAME=/usr/local/src/robot/r1-models/urdf/R1Mk3/robots/R1Mk3Gazebo

# WSLg + Mesa D3D12 OpenGL acceleration.
# These make the container prefer the WSLg D3D12 OpenGL path and select the
# NVIDIA adapter instead of llvmpipe or the integrated Intel adapter.
ENV XDG_RUNTIME_DIR=/mnt/wslg/runtime-dir
ENV LIBGL_ALWAYS_SOFTWARE=0
ENV GALLIUM_DRIVER=d3d12
ENV MESA_LOADER_DRIVER_OVERRIDE=d3d12
ENV MESA_D3D12_DEFAULT_ADAPTER_NAME=NVIDIA
ENV CYCLONEDDS_URI=/home/user1/.config/cyclone_dds_settings2.xml
RUN source /usr/local/src/robot/yarp-devices-ros2/ros2_interfaces_ws/install/setup.bash
RUN source /opt/ros/jazzy/setup.bash
ENV YARP_ROBOT_NAME=/usr/local/src/robot/r1-models/urdf/R1Mk3/robots/R1Mk3Gazebo
ENV GZ_SIM_RESOURCE_PATH="${GZ_SIM_RESOURCE_PATH}:/usr/local/src/robot/ycb-tools/models/ycb"

CMD ["/bin/bash"]
# Terminator default config
RUN mkdir -p /etc/xdg/terminator && \
    touch /etc/xdg/terminator/config

# Optional: make user1 the default user
USER user1
WORKDIR /usr/local/src/robot
