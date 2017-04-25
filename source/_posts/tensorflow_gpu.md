title: Ubuntu安装tensorflow_gpu
categories: 小笔记
date: 2016-08-29 17:54:21
tags: [Ubuntu,Tensorflow,GPU,Nvidia]
description: Ubuntu安装GPU版的tensorflow
---

## 硬件
对之前的实验训练速度终于忍无可忍了，剁手买了台主机回来，配置如下：
* CPU: intel i7 7700
* 主板: 技嘉B250（机箱自带的，略拙，应该搞个z270的）
* GPU: 技嘉 GTX 1070
* 内存: 賊船的复仇者单条16G DDR4

当时买的是整机。想来也是不考虑以后走多路了。

配置硬件前，看了撩机的资料。知乎里看到，配置的机器需要一张额外的显卡用来显示跑桌面的GUI（称点亮卡），另一张显卡用作计算用，理由时，显卡被GUI占用时无法进行cuda计算。所以，特别选了这个带CPU集显的。但事实测试使用时，只用独显带GUI和计算是完全没有问题的。不知道是不是以前的版本不行。当然也尝试过用集显点亮，用全部独显来计算，但总或多或少的有些问题。之后再测测吧，看能不能搞定。

## 软件
本来自己是被arch linux的pacman宠坏了，早用不习惯ubuntu了。但现在真懒得折腾了，所以，还是选择装ubuntu16.04。

### 驱动
看到不少人选择去nvidia上下载.run的来安装。不过，ubuntu上本来就有，而且版本并不比nvidia上的旧，所以直接从源里装了，但如果你是其他的一些发行版，估计还是要从官网下来装。
```
sudo apt-get update
sudo apt-get install -y nvidia-375 nvidia-settings
sudo apt-get install -y mesa-common-dev # 这里不装也没关系
sudo apt-get install -y freeglut3-dev   # 这里不装也没关系

# 如果想要切换，可以装nvidia-prime，之后使用prime-select intel切集显，当然，希望你的不会出我这里这咱启动不了的问题
sudo apt-get install -y nvidia-prime

# 如果加下面这个源的话，还可以安装更新的nvidia-381版的驱动
sudo add-apt-repository ppa:graphics-drivers/ppa

# 安装重启后可以用nvidia-smi查看状态
```

### cuda和cudnn
这两需要从Nvidia官网下载，而且现在下载它们还需要注册，填写调查表（不爽）。

cuda当时下载还是比较顺利的，但cudnn就奇怪的成龟速，加上代理都不行。后来是找到这个网址`http://developer.download.nvidia.com/compute/redist/cudnn/v6.0/cudnn-8.0-linux-x64-v6.0.tgz`，然后加到迅雷里下下来的。

```
# 安装cuda
sudo sh cuda_8.0.27_linux.run --tmpdir=/opt/temp/
# cuda安装中有一步是询问是否安装驱动，如下
# Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 ....? 
# 其实cuda包里是带了驱动的，但我们之前的安装更新，所以，这里选no
# 装好后请加上下面两个环境变量
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
# 之后便可以编译cuda的示例程序来测试了，比如`1_Utilities/deviceQuery`

# 安装cudnn
直接解压，将里面的内容复制到cuda的安装目录里即可。当然/usr/local/cuda可能是个软链接，比如我这里真正的目录是/usr/local/cuda-8.0
```

### 安装tensorflow_gpu
我比较喜欢anaconda这个全家桶，但conda源里没有python3.6的tensorflow-gpu。所以，直接用pip安装
```
pip install tensorflow_gpu
```

## 测试
```
import tensorflow as tf

input = tf.Variable(tf.random_normal([100, 28, 28, 1]))
filter = tf.Variable(tf.random_normal([5, 5, 1, 6]))

# 记得加上这个配置
sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))
tf.global_variables_initializer().run()

op = tf.nn.conv2d(input, filter, strides = [1, 1, 1, 1])
out = sess.run(op)
```

如果输出带有gpu字样，测安装成功。



