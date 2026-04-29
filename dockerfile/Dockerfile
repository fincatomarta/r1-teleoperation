# syntax=docker/dockerfile:1
FROM elandini84/r1images:tourSim2_ubuntu_24.04_qt_6.8.3_jazzy_devel

SHELL ["/bin/bash", "-lc"]
WORKDIR /usr/local/src/robot

RUN --mount=type=secret,id=github_token,env=GITHUB_TOKEN \
    git clone https://${GITHUB_TOKEN}@github.com/hsp-iit/grasping-baselines.git && \
    git clone https://${GITHUB_TOKEN}@github.com/hsp-iit/ergocub-cartesian-control.git

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
ENV LD_LIBRARY_PATH=/home/user1/robotology-superbuild/build/install/lib:$LD_LIBRARY_PATH
ENV YARP_DATA_DIRS=/home/user1/robotology-superbuild/build/install/share/yarp:$YARP_DATA_DIRS
ENV GZ_SIM_RESOURCE_PATH=$GZ_SIM_RESOURCE_PATH:/opt/ros/jazzy/share::/home/user1/installed/share:/home/user1/installed/share/R1Mk3/robots:/usr/local/src/robot/gz-sim-yarp-plugins/build/models:/usr/local/src/robot/LIRS-HMLG:/usr/local/src/robot/gz-sim-yarp-plugins/build/models:/usr/local/src/robot/LIRS-HMLG:/usr/local/src/robot/grasping-baselines/gazebo/models:/usr/local/src/robot/grasping-baselines/gazebo/worlds
ENV YARP_ROBOT_NAME=/usr/local/src/robot/r1-models/urdf/R1Mk3/robots/R1Mk3Gazebo
CMD ["/bin/bash"]
