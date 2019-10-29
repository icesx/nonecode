tensorflow
=============
##on ubuntu
###install
```
sudo apt install python python-pip
sudo apt install tensorflow #非gpu版本
```
###on raspberry
###install
```
sudo apt install python3-pip
#尽量用python3，因为python2 2020年就不维护了
pip3 install tensorflow
#如果安装过程出现 秘钥错误的问题，可以先wget下来，然后用如下命令安装
pip3 install tensorflow-1.13.1-cp35-none-linux_armv7l.whl
```
####illegal instruction (core dumped)
虚拟化软件中需要将cpu设置为Broadwell，不能用kvm-processer
###base test



###object_detection
```
mkdir ts
cd ts
git clone --recurse-submodules https://github.com/tensorflow/models.git
sudo nano ~/.bashrc
export PYTHONPATH=$PYTHONPATH:/home/pi/ts/models/research:/home/pi/ts/models/research/slim
cd /home/pi/ts/models/research
protoc object_detection/protos/*.proto --python_out=.
#This command converts all the "name".proto files to "name_pb2".py files
cd /home/pi/ts/models/research/object_detection
wget http://download.tensorflow.org/models/object_detection/ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz
tar -xzvf ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz
#The model zoo is Google’s collection of pre-trained object detection models that have various levels of speed and accuracy
wget https://raw.githubusercontent.com/EdjeElectronics/TensorFlow-Object-Detection-on-the-Raspberry-Pi/master/Object_detection_picamera.py
python3 Object_detection_picamera.py 
```
models/research/object_detection/

##编译安装
###prepare
1. 显卡驱动
NVIDIA-Linux-x86_64-430.40.run

```
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install libc6:i386
sudo bash -c "echo blacklist nouveau > /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
sudo bash -c "echo options nouveau modeset=0 >> /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
```

2. cuda 库（安装runtime和dev两个）
libcudnn7_7.6.2.24-1+cuda10.1_amd64.deb
libcudnn7-dev_7.6.2.24-1+cuda10.1_amd64.deb
3. 
pip3 install numpy 
###问题处理
####找不到cuda头文件
安装dev版本的cuda
####Illegal ambiguous match on configurable attribute “deps” in 
1. Do you want to use clang as CUDA compiler? [y/N] choose y, here you use clang as the backend cuda code compiler, then you can use command: bazel build --config=opt --config=cuda_clang //tensorflow/tools/pip_package:build_pip_package.

2. Do you want to use clang as CUDA compiler? [y/N] choose N, here you use nvcc as the backend cuda code compiler, then you can use command: bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package.