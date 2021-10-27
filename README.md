# Machine Learning Containers for Jetson and JetPack

Hosted on [NVIDIA GPU Cloud](https://ngc.nvidia.com/catalog/containers?orderBy=modifiedDESC&query=L4T&quickFilter=containers&filters=) (NGC) are the following Docker container images for machine learning on Jetson:

* [`l4t-ml`](https://ngc.nvidia.com/catalog/containers/nvidia:l4t-ml)
* [`l4t-pytorch`](https://ngc.nvidia.com/catalog/containers/nvidia:l4t-pytorch)
* [`l4t-tensorflow`](https://ngc.nvidia.com/catalog/containers/nvidia:l4t-tensorflow)

Dockerfiles are also provided for the following containers, which can be built for JetPack 4.4 or newer:

* ROS Melodic (`ros:melodic-ros-base-l4t-r32.4.4`)
* ROS Noetic (`ros:noetic-ros-base-l4t-r32.4.4`)
* ROS2 Eloquent (`ros:eloquent-ros-base-l4t-r32.4.4`)
* ROS2 Foxy (`ros:foxy-ros-base-l4t-r32.4.4`)

Below are the instructions to build and test the containers using the included Dockerfiles.

## Docker Default Runtime

To enable access to the CUDA compiler (nvcc) during `docker build` operations, add `"default-runtime": "nvidia"` to your `/etc/docker/daemon.json` configuration file before attempting to build the containers:

``` json
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    },

    "default-runtime": "nvidia"
}
```

You will then want to restart the Docker service or reboot your system before proceeding.

## Building the Containers

To rebuild the containers from a Jetson device running [JetPack 4.4](https://developer.nvidia.com/embedded/jetpack) or newer, first clone this repo:

``` bash
$ git clone https://github.com/dusty-nv/jetson-containers
$ cd jetson-containers
```

### ML Containers

To build the ML containers (`l4t-pytorch`, `l4t-tensorflow`, `l4t-ml`), use [`scripts/docker_build_ml.sh`](scripts/docker_build_ml.sh) - along with an optional argument of which container(s) to build: 

``` bash
$ ./scripts/docker_build_ml.sh all        # build all: l4t-pytorch, l4t-tensorflow, and l4t-ml
$ ./scripts/docker_build_ml.sh pytorch    # build only l4t-pytorch
$ ./scripts/docker_build_ml.sh tensorflow # build only l4t-tensorflow
```

> You have to build `l4t-pytorch` and `l4t-tensorflow` to build `l4t-ml`, because it uses those base containers in the multi-stage build.

Note that the TensorFlow and PyTorch pip wheel installers for aarch64 are automatically downloaded in the Dockerfiles from the [Jetson Zoo](https://elinux.org/Jetson_Zoo).

### ROS Containers

To build the ROS containers, use [`scripts/docker_build_ros.sh`](scripts/docker_build_ros.sh) with the name of the ROS distro to build:

``` bash
$ ./scripts/docker_build_ros.sh all       # build all: melodic, noetic, eloquent, foxy
$ ./scripts/docker_build_ros.sh melodic   # build only melodic
$ ./scripts/docker_build_ros.sh noetic    # build only noetic
$ ./scripts/docker_build_ros.sh eloquent  # build only eloquent
$ ./scripts/docker_build_ros.sh foxy      # build only foxy
```

Note that ROS Noetic and ROS2 Foxy are built from source for Ubuntu 18.04, while ROS Melodic and ROS2 Eloquent are installed from Debian packages into the containers.

## Testing the Containers

To run a series of automated tests on the packages installed in the containers, run the following from your `jetson-containers` directory:

``` bash
$ ./scripts/docker_test_ml.sh all        # test all: l4t-pytorch, l4t-tensorflow, and l4t-ml
$ ./scripts/docker_test_ml.sh pytorch    # test only l4t-pytorch
$ ./scripts/docker_test_ml.sh tensorflow # test only l4t-tensorflow
```

To test ROS:

``` bash
$ ./scripts/docker_test_ros.sh all       # test if the build of ROS all was successful: 'melodic', 'noetic', 'eloquent', 'foxy'
$ ./scripts/docker_test_ros.sh melodic   # test if the build of 'ROS melodic' was successful
$ ./scripts/docker_test_ros.sh noetic    # test if the build of 'ROS noetic' was successful
$ ./scripts/docker_test_ros.sh eloquent  # test if the build of 'ROS eloquent' was successful
$ ./scripts/docker_test_ros.sh foxy      # test if the build of 'ROS foxy' was successful
```
## Before ros noetic container runs
Clone this repository and clone "https://github.com/NTNU-aFerry/ros_af_pytorch_detector" and "https://github.com/NTNU-aFerry/ros_af_msgs". Go back to the "jetson-container" directory and use the commands "cp /etc/apt/trusted.gpg.d/jetson-ota-public.asc ." and "cp /etc/apt/sources.list.d/nvidia-l4t-apt-source.list ."


## Running the ros noetic container 

```
./scripts/docker_run.sh -c ros:noetic-ros-base-l4t-r32.5.1
```
Then inside workspace/ run:
```
# Build tools for cv_bridge
apt-get update && apt-get install -y python3-pip python3-yaml
pip3 install rospkg catkin_pkg
apt-get update && apt-get install -y python-catkin-tools python3-dev python3-numpy
apt-get install vim
```

and then follow the setup.bash in (but changing to noetic) https://github.com/NTNU-aFerry/ros_af_pytorch_detector/blob/master/docker/workspace/setup.sh
```
# Source ROS noetic
source /opt/ros/noetic/setup.bash

# Create workspaces that are needed.
mkdir -p aferry_ws/src
mkdir -p catkin_build_ws/src

# Build cv_bridge
cd catkin_build_ws
catkin config -DPYTHON_EXECUTABLE=/usr/bin/python3 -DPYTHON_INCLUDE_DIR=/usr/include/python3.7m -DPYTHON_LIBRARY=/usr/lib/x86_64-linux-gnu/libpython3.7m.so
catkin config --install

cd src
git clone -b noetic https://github.com/ros-perception/vision_opencv.git
cd cv_bridge
vim CMakeLists.txt
# change Boost REQUIRED python 37 to python 3
#and change: "find_package(OpenCV ${_opencv_version} REQUIRED" to "find_package(OpenCV 4 REQUIRED"

cd ~/workspace/catkin_build_ws
catkin build cv_bridge
pwd

source install/setup.bash --extend
cd ../aferry_ws
python3 -m venv --system-site-packages ./venv

source venv/bin/activate
pip install -U pip
cd src
git clone https://github.com/Skarsh/yaep_lib.git
cd yaep_lib && pip install -e .

echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
echo "source /workspace/aferry_ws/venv/bin/activate" >> ~/.bashrc
```
