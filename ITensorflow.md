tensorflow
=============
## on ubuntu
### install

```
sudo apt install python3 python3-pip
pip3 install tensorflow 
#非gpu版本
```
### 模型训练



## on raspberry

### install

```
sudo apt install python3-pip
#尽量用python3，因为python2 2020年就不维护了
pip3 install tensorflow
#如果安装过程出现 秘钥错误的问题，可以先wget下来，然后用如下命令安装
pip3 install tensorflow-1.13.1-cp35-none-linux_armv7l.whl
```
#### illegal instruction (core dumped)
虚拟化软件中需要将cpu设置为Broadwell，不能用kvm-processer

### base test



### object_detection

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

## 编译安装
### prepare

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
3. pip3 install numpy 
### 问题处理
#### 找不到cuda头文件
安装dev版本的cuda

#### Illegal ambiguous match on configurable attribute “deps” in 
1. Do you want to use clang as CUDA compiler? [y/N] choose y, here you use clang as the backend cuda code compiler, then you can use command: bazel build --config=opt --config=cuda_clang //tensorflow/tools/pip_package:build_pip_package.

2. Do you want to use clang as CUDA compiler? [y/N] choose N, here you use nvcc as the backend cuda code compiler, then you can use command: bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package.

## pip 安装

```

```



## Tensorflow2.x

tf2的版本去掉了session，另外还有较大的改动。官方推荐使用keras进行开发。

### 常用层

1. Flatten层

> Flatten层用来将输入“压平”，即把多维的输入一维化，常用在从卷积层到全连接层的过渡。Flatten不影响batch的大小。

2. Dense

> Dense就是常用的全连接层，所实现的运算是`output = activation(dot(input, kernel)+bias)`。其中`activation`是逐元素计算的激活函数，`kernel`是本层的权值矩阵，`bias`为偏置向量，只有当`use_bias=True`才会添加。

3. Activation层

> 激活层对一个层的输出施加激活函数

4. Dropout

> Dropout将在训练过程中每次更新参数时按一定概率（rate）随机断开输入神经元，Dropout层用于防止过拟合

5. Reshape

> Reshape层用来将输入shape转换为特定的shape

6. Permute
>Permute层将输入的维度按照给定模式进行重排，例如，当需要将RNN和CNN网络连接时，可能会用到该层。

### 卷积层

1. Conv1D

> 一维卷积层（即时域卷积），用以在一维输入信号上进行邻域滤波

2. Conv2D

> 二维卷积层，即对图像的空域卷积

3. ​	

### 几个模型训练的参数

batchsize：中文翻译为批大小（批尺寸）。在深度学习中，一般采用SGD训练，即每次训练在训练集中取batchsize个样本训练；
iteration：中文翻译为迭代，1个iteration等于使用batchsize个样本训练一次；
一个迭代 = 一个正向通过+一个反向通过
epoch：迭代次数，1个epoch等于使用训练集中的全部样本训练一次；
一个epoch = 所有训练样本的一个正向传递和一个反向传递
举个例子，训练集有1000个样本，batchsize=10，那么：
训练完整个样本集需要：
100次iteration，1次epoch。