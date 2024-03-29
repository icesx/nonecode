IML
===

## CRF
### 条件随机场（CRF）
	给定一组输入随机变量的条件下另一组输出随机变量的条件概率分布

## HMM

### 基本要素

1. A:状态转移概率矩阵 
   len(pi)*len(pi) 矩阵
   t时刻状态q_i的条件下在时刻t+1转移到状态q_j的概率

2. B:发射概率矩阵【也称为观测概率矩阵】 
   t时刻处于状态q_j的条件下生成观测v_k的概率
   在维特比算法中称之为发射概率

3. pi:初始状态向量 
   所有可能出现的状态的组成的向量

状态转移概率矩阵A与初始状态概率向量pi确定了隐藏的马尔科夫链，生成不可观测的状态序列。观测概率矩阵B确定了如何从状态生成观测，与状态序列综合确定了如何产生观测序列
	
### 相关概念

1. 观测向量或者观测序列
   StatusSet: 状态值集合
   ObservedSet: 观察值集合

### 模型训练
	HMM训练的目标是产生两个分布：状态转移矩阵和发射概率矩阵
	HMM存在两种训练的方法，第一种为监督学习：训练样本中存在标注信息；第二种为非监督学习：训练样本中不存在标注信息。
#### 监督学习
	1. 转移概率矩阵估计

## 开源数据集

### text recognation 

https://www.robots.ox.ac.uk/~vgg/data/text/

### 人脸识别

![img](/ICESX/ISunflower/nonecode/IML.assets/03b0616ebf7b03d26dc4d52b9da8807e.png)

## 参考模型

### AlexNet

![image-20220209214805515](/ICESX/ISunflower/nonecode/IML.assets/image-20220209214805515.png)

### VGG

![CNN结构演变总结（一）经典模型](/ICESX/ISunflower/nonecode/IML.assets/29102354628639.png)

## 资源

https://wizardforcel.gitbooks.io/dm-algo-top10/content/cart.html
