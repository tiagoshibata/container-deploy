FROM nvidia/cuda:8.0-cudnn5-devel-ubuntu16.04
MAINTAINER Tiago Koji Castro Shibata


# General setup
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
RUN apt-get update && apt-get install --no-install-recommends -y build-essential sudo \
    && rm -rf /var/lib/apt/lists/*
# Configure PATH for all users
RUN echo PATH="/usr/local/nvidia/bin:/usr/local/cuda/bin:$PATH" > /etc/profile.d/nvidia-path.sh


# ROS
RUN apt-key adv --keyserver ha.pool.sks-keyservers.net --recv-keys 421C365BD9FF1F717815A3895523BAEEB01FA116 \
    && echo 'deb http://packages.ros.org/ros/ubuntu xenial main' > /etc/apt/sources.list.d/ros-latest.list
ENV ROS_DISTRO kinetic
RUN apt-get update && apt-get install --no-install-recommends -y cmake ros-kinetic-desktop-full python-wstool \
    && rm -rf /var/lib/apt/lists/*
RUN useradd -mG sudo ros && echo source "/opt/ros/$ROS_DISTRO/setup.bash" >> ~ros/.bashrc \
    && mkdir -p /etc/sudoers.d && echo '%sudo   ALL=(ALL:ALL) NOPASSWD: ALL' > /etc/sudoers.d/sudo_nopasswd


# Caffe deps
RUN apt-get update && apt-get install --no-install-recommends -y git gfortran \
    libatlas-base-dev libboost-all-dev libgflags-dev libgoogle-glog-dev libhdf5-serial-dev \
    libleveldb-dev liblmdb-dev libprotobuf-dev libsnappy-dev protobuf-compiler \
    python-all-dev python-dev python-h5py python-matplotlib python-numpy \
    python-pil python-pip python-protobuf python-scipy python-skimage python-sklearn \
    && rm -rf /var/lib/apt/lists/*
# OpenCV w/ CUDA
RUN git clone https://github.com/opencv/opencv.git ~/opencv && cd ~/opencv \
    && git reset --hard 10896129b39655e19e4e7c529153cb5c2191a1db
RUN cd ~/opencv && mkdir build && cd build && cmake -DBUILD_DOCS=OFF -DBUILD_PERF_TESTS=OFF \
    -DBUILD_TESTS=OFF -DCMAKE_BUILD_TYPE=RELEASE -DCUDA_ARCH_BIN="3.0 3.5 5.0 6.1" \
    -DCUDA_ARCH_PTX="3.0" -DCUDA_FAST_MATH=ON -DENABLE_FAST_MATH=ON -DENABLE_SSE41=ON \
    -DENABLE_SSE42=ON -DENABLE_SSSE3=ON -DWITH_CUBLAS=ON -DWITH_MATLAB=OFF .. \
    && make -j4 && make install && rm -rf ~/opencv*
# NVIDIA CUB
RUN git clone --depth=5 https://github.com/NVlabs/cub.git ~/cub && cd ~/cub \
    && git checkout 1.5.5 && cp -r cub /usr/local/include/cub


# Caffe
RUN git clone --depth=5 https://github.com/lunzueta/caffe ~/caffe && cd ~/caffe \
    && git reset --hard abed522996d309e948e3f142454885d333704358
RUN pip install -U pip && pip install -r ~/caffe/python/requirements.txt && rm -rf ~/.cache/pip
RUN mkdir -p ~/caffe/build && cd ~/caffe/build && cmake -DBUILD_docs=OFF -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/local -DOpenCV_DIR=/usr/local/share/OpenCV .. \
    && make -j4 all && make install && rm -rf ~/caffe
RUN echo export PYTHONPATH=/usr/local/python > /etc/profile.d/python-usr-local.sh


# RTIMULib2
RUN git clone --depth=5 https://github.com/richardstechnotes/RTIMULib2.git ~/RTIMULib2
RUN mkdir ~/RTIMULib2/Linux/build && cd ~/RTIMULib2/Linux/build && cmake -DBUILD_GL=OFF .. \
    && make -j4 && sudo make install && rm -rf ~/RTIMULib2


# Other packages: gpsd, v4l-utils (for v4l2-ctl)
RUN apt-get update && apt-get install --no-install-recommends -y gpsd libgps-dev v4l-utils \
    && rm -rf /var/lib/apt/lists/*


COPY ./run.sh /home/ros/
COPY ./ros_entrypoint.sh /
ENTRYPOINT ["/ros_entrypoint.sh"]
CMD ["/home/ros/run.sh"]