tensorflow
=============
## on ubuntu
### install

```sh
sudo apt install python3 python3-pip
pip3 install tensorflow 
#非gpu版本
```
### 模型训练



## on raspberry

### install

```sh
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

```sh
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

1. 下载显卡驱动
NVIDIA-Linux-x86_64-430.40.run

```
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install libc6:i386
sudo bash -c "echo blacklist nouveau > /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
sudo bash -c "echo options nouveau modeset=0 >> /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
```

2. cuda安装

  从novida[官网](https://developer.nvidia.com/cuda-toolkit-archive)下载并按照说明安装cuda

3. cudann 库（安装runtime和dev两个）
    libcudnn7_7.6.2.24-1+cuda10.1_amd64.deb
    libcudnn7-dev_7.6.2.24-1+cuda10.1_amd64.deb

4. pip3 install numpy 
### 问题处理
#### 1. 找不到cuda头文件
安装dev版本的cuda

#### Illegal ambiguous match on configurable attribute “deps” in 
1. Do you want to use clang as CUDA compiler? [y/N] choose y, here you use clang as the backend cuda code compiler, then you can use command: bazel build --config=opt --config=cuda_clang //tensorflow/tools/pip_package:build_pip_package.

2. Do you want to use clang as CUDA compiler? [y/N] choose N, here you use nvcc as the backend cuda code compiler, then you can use command: bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package.

#### 2. libcudart.so 

> 没有安装cuda

## pip 安装

```
pip install tensorflow-gpu
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

1. batchsize

   > 中文翻译为批大小（批尺寸）。在深度学习中，一般采用SGD训练，即每次训练在训练集中取batchsize个样本训练；

2. iteration

   > 中文翻译为迭代，1个iteration等于使用batchsize个样本训练一次；
   > 一个迭代 = 一个正向通过+一个反向通过

3. epoch

   > 迭代次数，1个epoch等于使用训练集中的全部样本训练一次；
   > 一个epoch = 所有训练样本的一个正向传递和一个反向传递
   > 举个例子，训练集有1000个样本，batchsize=10，那么：
   > 训练完整个样本集需要：
   > 100次iteration，1次epoch。



## 训练模型

如果出现损失函数为nan的情况，可能存在如下问题

1. 数据有问题，数据与当前的模型不匹配
2. label的id没有从0开始，导致设置了分类为10个，最大的label的id是10。

## 欠拟合

### 判断

1. accuracy 降低，而validation增加

2. accuracy 不降低

   > 检查steps_per_epoch*batch是不是<=train样本的数量。如果大于的话，可能有样本被重复训练了

## 过拟合

### 判断

1. 如果validation的loss增加而train的loss减少，应该是过拟合了。
2. accuracy快速上升，一般可能过拟合了。
3. 

### 如何处理

1. 获得更多的训练数据

   > 可以使用样本增强augment

2. 缩小网络规模

   > 降低参数个数

1. 权重正则化

   >  增加L2正则

2. Dropout 

### 防止

**他们一般是在小的数据集上线训练小网络，确定这种结构work之后再在更大的数据集上train更深的网络**。

## 关于卷积核

> 卷积核大小决定了一个神经元感受野的大小，当卷积核过小的时候，无法提取有效的局部特征，当卷积核过大的是呵呵，提取的特征的复杂度可能远远超过卷积核的表示能力。



## 关于训练

# train loss与test loss结果分析

train loss 不断下降，test loss不断下降，说明网络仍在学习;
train loss 不断下降，test loss趋于不变，说明网络过拟合;
train loss 趋于不变，test loss不断下降，说明数据集100%有问题;
train loss 趋于不变，test loss趋于不变，说明学习遇到瓶颈，需要减小学习率或批量数目;
train loss 不断上升，test loss不断上升，说明网络结构设计不当，训练超参数设置不当，数据集经过清洗等问题。



### 还可以作的优化

1. 池化层步长修改为2
2. batch之前进行shffer



### 关于学习率

数用来控制学习率减缓幅度,t 为训练轮数(2TQ+?)
除此之外,寻找理想学习率或诊断模型训练学习率是否合适时可借助模型训
练曲线(H2`MBM; +m`p2)的帮助。训练深度网络时不妨将每轮训练后模型在目
标函数上的损失值保存,以图 RRXk所示形式画出其训练曲线。读者可将自己的
训练曲线与图中曲线“对号入座”:若模型损失值在模型训练刚开始的几个批次
直接“爆炸”(黄色曲线),则学习率过大,此时应大幅减小学习率从头训练网
络;若模型一开始损失值下降明显,但“后劲不足”(绿色曲线),此时应使用
较小学习率从头训练,或在后几轮改小学习率仅重新训练后几轮即可;若模型
损失值一直下降缓慢(蓝色曲线),此时应稍微加大学习率,然后继续观察训练
曲线;直至模型呈现红色曲线所示的理想学习率下的训练曲线为止。此外,微
调(}M2 imM2)卷积神经网络过程中,学习率有时也需特别关注