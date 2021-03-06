# Ubuntu-18.04-with-Opencv-and-Tensorflow
This tutorial walks you through installing OpenCV and Tensorflow (Python and C++) with/without CUDA support in Ubuntu-18.04. 

## Update Ubuntu
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get autoremove
```
## Install Required Libraries
```
sudo apt-get install openjdk-8-jdk git python3-dev python3-numpy python3-six build-essential python3-pip swig python3-wheel libcurl3-dev libcupti-dev
```
## Install Nvidia Drivers For GPU
*You may skip this step if you do not have a GPU*

Add graphics to your source list. Then update and upgarde
```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
sudo apt upgrade
```
 Auto-install the latest drivers
 ```
 sudo ubuntu-drivers autoinstall
 ```

Reboot your machine and check if the correct version of the drivers has been installed using the following commands
```
sudo reboot

nvidia-smi
```

## Install CUDA Drivers
*You may skip this step if you do not have a GPU*

At the time of writing this blog **CUDA 10** for Ubuntu 18.04 is available. However, since the latest Tensorflow version (1.10) is not stable with CUDA 10, I will be using **CUDA 9.0 with CUDADNN 7.1**. Kindly use the latest stable version of CUDA and cuDNN available at the time of install.

You can download and install the latest CUDA_VERSION.run file from https://developer.nvidia.com/cuda-toolkit. After downoading, go the Downloads directory and install using the following command:
```
cd Downloads/
sudo sh cuda_9.0.176_384.81_linux.run --override --silent --toolkit
```

You can download and install the latest cuDNN_VERSION file from https://developer.nvidia.com/cudnn. After you login with your NVIDIA developer credentials, kindly download the linux version of the cuDNN version since cuDNN 7.1.4 for Ubuntu 18.04 is not yet available . After downloading, go the *Downloads* directory and install cuDNN using the following command:
```
tar -xzvf cudnn-9.0-linux-x64-v7.1.tgz 
sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
```
Open /.bashrc script
```
gedit ~/.bashrc
```
Add the following path to the end of your /.bashrc script
```
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64"
export CUDA_HOME=/usr/local/cuda
```
Refresh and reload terminal config
```
source ~/.bashrc
sudo ldconfig
```
Check for correct installation using the following command
```
echo $CUDA_HOME
```
## Install OpenCV
### Install Cmake
```
sudo apt-get install build-essential cmake
```
### Additional Dependencies

```
sudo apt-get install qt5-default libvtk6-dev
```
### Download OpenCV
For this tutorial, we will be using **OpenCV 3.4.3**
```
git clone https://github.com/opencv/opencv.git
cd opencv
git checkout 3.4.3
cd ..
```
### Download OpenCV Contrib
```
git clone https://github.com/opencv/opencv_contrib.git
cd opencv_contrib
git checkout 3.4.3
cd ..
```
### Configure and Generate Make File
```
cd opencv
mkdir build
cd build
```
*For Compiling OpenCV without CUDA support*
```
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D WITH_QT=ON \
      -D WITH_OPENGL=ON \
      -D WITH_TBB=ON \
      -D WITH_GDAL=ON \
      -D BUILD_EXAMPLES=ON..
```

*For Compiling OpenCV with CUDA support*

```
cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D FORCE_VTK=ON \
-D WITH_TBB=ON \
-D WITH_V4L=ON \
-D WITH_QT=ON \
-D WITH_OPENGL=ON \
-D WITH_CUBLAS=ON \
-D CUDA_NVCC_FLAGS="-D_FORCE_INLINES" \
-D WITH_GDAL=ON \
-D WITH_XINE=ON \
-D WITH_CUDA=ON\
-D BUILD_EXAMPLES=ON ..

```

```
make -j4
sudo make install
sudo ldconfig
```
## Install Tensorflow with Python3 (And GPU support)
### Install Dependencies
```
pip3 install numpy scipy matplotlib scikit-image scikit-learn ipython protobuf jupyter pandas

```
### Install Tensorflow (CPU)
```
pip3 install tensorflow
```
### Install Tensorflow (GPU - Optional)
```
pip3 install tensorflow-gpu 
```
### Install Dependencies
Install Theano, Keras and Dlib
```
pip3 install Theano 
pip3 install keras
pip3 install dlib
```
*Installing PyTorch*
The following command must redirect you to the official PyTorch Website. 
```
pip3 install pytorch
```
You can choose the appropriate command to install PyTorch

## Check Your Installation
If you have installed  everything correctly until this point, your output must look similar to this

## Install Tensorflow (C++) with Bazel
### Install gcc and g++
Ubuntu 18.04 comes with built-in gcc and g++. If you are using an older version of Ubuntu, then install gcc and g++ using the following commands
```
sudo apt-get install gcc g++
```
### Install Bazel
```
sudo apt install curl
echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
sudo apt-get update
sudo apt-get install bazel
sudo apt-get upgrade bazel
```
### Download Tensorflow
For this tutorial we use Tensorflow 1.10. 
```
git clone https://github.com/tensorflow/tensorflow
cd tensorflow
git checkout r1.10
```
### Create Configuration File
```
./configure
```
### Build Tensorflow with Bazel
*Install Bazel with C++ without CUDA Support*
```
bazel build --jobs=6 --verbose_failures -c opt --copt=-mavx --copt=-mfpmath=both --copt=-msse4.2 //tensorflow:libtensorflow_cc.so
```

*Install Bazel with C++ with CUDA Support*
```
bazel build -c opt --copt=-mavx2 --copt=-mfma --copt=-mfpmath=both --copt=-msse4.2 --config=cuda --verbose_failures //tensorflow:libtensorflow_cc.so
```

```
bazel build -c opt --copt=-mavx2 --copt=-mfma --copt=-mfpmath=both --copt=-msse4.2 --config=cuda --verbose_failures //tensorflow/tools/pip_package:build_pip_package
```
