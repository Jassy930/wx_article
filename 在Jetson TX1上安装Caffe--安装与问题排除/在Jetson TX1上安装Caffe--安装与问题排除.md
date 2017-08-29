[TOC]

>最近想在Jetson TX1这个板子上学习Caffe和机器学习一些内容，于是把安装Caffe的过程和其中遇到的问题做一个学习笔记。
>首先正常的安装可以参考Caffe的官网和网上数不清的教程，这里不再记录了，重点说安装中出现的问题与解决办法。

__________
### 一、在安装Caffe之前需要进行依赖项的安装：

这里很多地方都讲到了，需要安装一些依赖库，下面是找到的比较全的

```
sudo apt-get install libprotobuf-dev protobuf-compiler gfortran \
libboost-dev cmake libleveldb-dev libsnappy-dev libopencv-dev\
libboost-thread-dev libboost-system-dev \
libatlas-base-dev libhdf5-serial-dev libgflags-dev \
libgoogle-glog-dev liblmdb-dev gcc-4.7 g++-4.7
```
但是在这其中中在给JetsonTX1安装libopencv-dev时会无法安装，因为安装了libopencv4tegra，这个是NVDIA官方给板子已经装的，所以一般情况下这个包不需要进行安装直接跳过无视错误信息就好。

_______
### 二、下载Caffe源码并对其进行编译

这里建议参考

>[Tegra TX1 安装配置 + caffe run](http://blog.csdn.net/mydear_11000/article/details/51968065)

这篇文章讲到需要使用g++4.7进行编译，因为默认使用的g++4.8在编译时会发生错误，我第一次并没有看到这篇文章，所以也理所当然的出错了。

__________
### 三、Caffe编译错误 nvcc fatal : Unsupported gpu architecture 'compute_60'

caffe编译时出现错误：
```
nvcc fatal : Unsupported gpu architecture 'compute_60'
```
在网上找了很多资料也没有直接说明白这个问题的解决办法的，因为关于JetsonTX1的资料太少，后来在这篇文章

>[【matconvnet】故障排除：Error using mex nvcc fatal : Unsupported gpu architecture 'compute_52'](http://blog.csdn.net/u013657981/article/details/50519123)

中找到了一些端倪，大概定位了错误的问题。 这个问题起源于编译器无法识别程序中的'compute_60'，编译器是CUDA的nvcc，于是我尝试去升级CUDA的版本，感觉可以解决这个问题。但是没有找到JetsonTX1装CUDA7.0以上版本的例子，作为初学者不敢大动，于是又回去翻看上面这篇文章。想到既然是编译中出现的错误，肯定也可以自己去定位错误看是否可以找到问题改正。在看出错文件的时候突然想到Caffe编译时是有一个Makefile.config文件的！！！立马去看这个文件，看到了下面几行内容：
```
# CUDA architecture setting: going with all of them.
# For CUDA < 6.0, comment the *_50 through *_61 lines for compatibility.
# For CUDA < 8.0, comment the *_60 and *_61 lines for compatibility.
CUDA_ARCH := -gencode arch=compute_20,code=sm_20 \
-gencode arch=compute_20,code=sm_21 \
-gencode arch=compute_30,code=sm_30 \
-gencode arch=compute_35,code=sm_35 \
-gencode arch=compute_50,code=sm_50 \
-gencode arch=compute_52,code=sm_52 \
-gencode arch=compute_60,code=sm_60 \
-gencode arch=compute_61,code=sm_61 \
-gencode arch=compute_61,code=compute_61
```
立马感觉自己真的是蠢到家了，在配置文件中写的如此清楚明了，CUDA7.0版本将最后三行注释掉，Caffe顺利编译通过。

_______
### 四、基准测试代码运行错误

本来到上面那里这篇应该是结束了，但是Caffe有测试的例子于是决定跑一下看看计算速度怎么样，也验证一下安装是否确实成功了，这个时候出现了另外一个问题。当使用CPU去跑的时候可以顺利跑起来（速度也惨不忍睹，一次迭代9000+ms)。但是就是无法用GPU进行运算，会出现以下错误：
```
Check failed: error == cudaSuccess (8 vs. 0) invalid device function
```
瞬间感觉有点崩，过程太坎坷。不过这个问题解决没有花费多少时间，上网搜索找到了这篇文章：

>[caffe问题及解决方法](http://blog.csdn.net/cwt19902010/article/details/49201333)

其中提到了相同的问题，同样是因为上面那一块的config的问题，这时候想起来很早的时候看到NVDIA的计算能力的一个表格，嗯，是这个：

>[CUDA GPUs](https://developer.nvidia.com/cuda-gpus)

在这里查到Jetson TX1的计算能力为5.3于是在上面那块内容中再加入一行：
```
-gencode arch=compute_53,code=sm_53 \
```

问题顺利解决

_______
### 五、测试与资料

最后都装完了，就放一下测试最后的结果吧。还可以哈哈
```
Average Forward pass: 175.87 ms.
Average Backward pass: 144.709 ms.
Average Forward-Backward: 321.06 ms.
Total Time: 16053 ms.
```
然后这里Mark一下，Caffe资料汇总，后面可能需要看的

http://blog.csdn.net/langb2014/article/details/51543388

>吐槽一下，这个文章的编辑太消耗精力了，我要去看看wordpress能不能用markdown来写

>赞美Markdown，简直好用，现在这篇文章已经改为使用Sublime3,Markdown编写，转为HTML格式发布的了