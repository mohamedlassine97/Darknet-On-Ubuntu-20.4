# Darknet-On-Ubuntu-20.4
Installing  YOLO Darknet  with GPU, CUDA, cuDNN and openCV  on  ubuntu 20.04

## 1. Cleaning up
Open a terminal window and type the following three commands to get rid of  
any NVIDIA/CUDA packages you may already have installed  
```bash
sudo rm /etc/apt/sources.list.d/cuda*
sudo apt remove --autoremove nvidia-cuda-toolkit
sudo apt remove --autoremove nvidia-*
```
Purge any remaining NVIDIA configuration files and the associated dependencies  
that they may have been installed with.
```bash
sudo apt-get purge nvidia*
sudo apt-get autoremove
sudo apt-get autoclean
```
 Remove any existing CUDA folders you may have in **/usr/local/**
 ```bash
 sudo rm -rf /usr/local/cuda*
 ```
 ## 2. Install
 Setting up the CUDA PPA.  About [PPA in ubuntu](https://itsfoss.com/ppa-guide/).  
 CUDA is added to source.list file.  
```bash
sudo apt update
sudo add-apt-repository ppa:graphics-drivers
sudo apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda.list'
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda_learn.list'
```
Install cuda 10.1 packages
```bash
sudo apt update
sudo apt install cuda-10-1
```
>**Note:**  
>&nbsp;&nbsp;&nbsp;  While installing cuda-10.1, apt manager may install libcublas package for cuda-10.2 instead of  
>&nbsp;&nbsp;&nbsp;  the one compatible with cuda 10.1, which can cause issues when using gpu functionality.  

To avoid this problem, let's remove this package and install the compatible package for cuda-10.1.  
```bash
# remove libcublas10.2
sudo apt-get remove libcublas10* 

# download cublas runtime and developper libraries (.deb)
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/libcublas10_10.1.0.105-1_amd64.deb      
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/libcublas-dev_10.1.0.105-1_amd64.deb

# install (.deb) files
sudo dpkg -i libcublas10_10.1.0.105-1_amd64.deb
sudo dpkg -i libcublas-dev_10.1.0.105-1_amd64.deb
```

```diff
+ After installing CUDA we reboot the system
```
To install cuDNN library we download cuDNN Runtime, Developper and and Code Samples int (.deb)  format from [here](http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64/).  
Download -->  cuDNN Runtime Library  for cuda 10.1  
Download -->  cuDNN Developper Library  for cuda 10.1   
Download -->  cuDNN Code Samples and User Guide  for cuda 10.1  
Depackage and Install downloaded packages
```bash
sudo dpkg -i libcudnn7_7.6.5.32-1+cuda10.1_amd64.deb
sudo dpkg -i libcudnn7-dev_7.6.5.32-1+cuda10.1_amd64.deb
sudu dpkg -i libcudnn7-doc_7.6.5.32-1+cuda10.1_amd64.deb
```
Our cuda file should be in **/usr/local** directory as ***cuda-10.1***  
but compile Makefile knows it as cuda. So we should rename this directory as ***cuda***  
or create a new directory as ***cuda*** and copy ***cuda-10.1*** to this one.  
Another option is to create a symlink as **cuda** to ***cuda-10.1*** directory.  
```bash
# 1. Option: rename the directory
sudo mv /usr/local/cuda-10.1  /usr/local/cuda

# 2. Option: create new directory
sudo mkdir /usr/local/cuda
sudo cp -r /usr/local/cuda-10.1/*  /usr/local/cuda/

# 3. Option: create a symlink to the directory
sudo ln -s /usr/local/cuda-10.1  /usr/local/cuda
```
To complete the setup, we download the cuDNN library files and move them to the appropriate directories.  

```bash
wget https://developer.nvidia.com/compute/machine-learning/cudnn/secure/7.6.5.32/Production/10.1_20191031/cudnn-10.1-linux-x64-v7.6.5.32.tgz
tar -xzf cudnn-10.1-linux-x64-v7.6.5.32.tgz
cd cuda
sudo cp include/cudnn.h /usr/local/cuda/include
sudo cp lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
```

After installing, let's open our **.profile** file and add this short script using vim.  
```
# open .profile
sudo vim ~/.profile

# add this script and save
if [ -d "/usr/local/cuda/bin/" ]; then
    export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
    export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
fi
```
So to activate it, let's run this command
```
source ~/.profile
```
As we just added the CUDA path to our **.profile**, now the shell knows where to find CUDA.  

<br />

Now let's verify our cuDNN installation, this is **optional**
```bash
cp -r /usr/src/cudnn_samples_v7/ $HOME
cd $HOME/cudnn_samples_v7/mnistCUDNN
make clean && make
./mnistCUDNN
```
Let's verify some installations
```bash
python3 --version
```
if not installed
```bash
sudo apt-get install python3
```

```bash
pip3 --version
```
if not installed
```bash
sudo apt-get install python3-pip
python3 -m pip install --upgrade pip
```

```bash
gcc --version
```
if not installed or version >= 8  
we should install one or multiple versions and choose the compatible one via **update-alternatives**  
To do this let's follow these steps  

#### 1. Start by updating packages
```bash
sudo apt update
```

#### 2. Install the build-essential package by typing:
```bash
sudo apt install build-essential
```

#### 3. Install manual pages (optional)
```bash
sudo apt-get install manpages-dev
```

#### 4. The default version
```bash
gcc --version
```
#### 5. Install multiple gcc versions and set the one we want as default
- First, add the ubuntu-toolchain-r/test PPA to your system with:
```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
```
- Install the desired GCC and G++ versions by typing:
```bash
sudo apt install gcc-7 g++-7 gcc-8 g++-8 gcc-9 g++-9
```
- Configure alternative for each version. The default one is with the highest priority
```bash
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 90 --slave /usr/bin/g++ g++ /usr/bin/g++-9 --slave /usr/bin/gcov gcov /usr/bin/gcov-9
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 80 --slave /usr/bin/g++ g++ /usr/bin/g++-8 --slave /usr/bin/gcov gcov /usr/bin/gcov-8
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 70 --slave /usr/bin/g++ g++ /usr/bin/g++-7 --slave /usr/bin/gcov gcov /usr/bin
```
- Change the default one
```bash
sudo update-alternatives --config gcc
```
- Output
```
There are 3 choices for the alternative gcc (providing /usr/bin/gcc).

  Selection    Path            Priority   Status
------------------------------------------------------------
* 0            /usr/bin/gcc-9   90        auto mode
  1            /usr/bin/gcc-7   70        manual mode
  2            /usr/bin/gcc-8   80        manual mode
  3            /usr/bin/gcc-9   90        manual mode

Press <enter> to keep the current choice[*], or type selection number:

```
We enter the ID of the one we want to be set as default and press enter  
<br>

Instaling openCV
```bash
sudo apt install python3-opencv libopencv-dev libopencv-core-dev
pip3 install opencv-python
```
Verify the installation
```
python3 -c "import cv2; print (cv2.__version__)"
```

## Darknet
- Clone darknet repository
```bash
git clone https://github.com/AlexeyAB/darknet.git
```
- Install using Cmake
```bash
cd darknet
./build.sh
```
- Install with GPU, CUDA, cuDNN and openCV via the MakeFile
Let's open the make file and make some changes
in the first lines we have variables as  
```make
GPU=0
CUDNN=0
CUDNN_HALF=0
OPENCV=0
```
- We change this values to  
```make
GPU=1
CUDNN=1
CUDNN_HALF=1
OPENCV=1
```
- In order to enable CUDA cudNN and openCV  
Then we save the file and open the terminal window in the darknet core directory  
```
make
```
- Then once the compilation finished we verify the installation by typing {} in te terminal
```
./darknet
```
- If all goes well, we should have as output:
```
usage: ./darknet <function>
```
- Test on an image
```
./darknet detector test cfg/coco.data cfg/yolov3.cfg weights/yolov3.weights data/test-image.jpg
```

- Details:  
  
```diff
- data/test_image.jpg: Predicted in 61.536000 milli-seconds.  
+ chair: 71% 
+ person: 100% 
+ tvmonitor: 99% 
+ keyboard: 27% 
+ pottedplant: 43%
+ book: 58%
+ book: 31% 
+ book: 26%
+ book: 72%
+ book: 56% 
+ book: 75% 
+ book: 69% 
+ book: 93% 
+ pottedplant: 91% 
+ vase: 29% 
```


<p align=center>
<img src="test_image.jpg" alt="Test Image" title="Test Image"  width="1000">
</p>

