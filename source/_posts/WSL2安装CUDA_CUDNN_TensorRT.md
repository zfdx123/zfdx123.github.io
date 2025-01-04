---
layout: post
title: WSL2 环境下配置CUDA和CUDNN并部署TensorRT
date: 2024-12-22 12:00:00
tags: 
    - AI
    - WSL
categories: 
    - AI
    - WSL
thumbnail: "images/wsl_cuda/logo.png"
cover: "images/wsl_cuda/logo.png"
license: cc_by_nc_sa
---

## CUDA

[cuda下载地址](https://developer.nvidia.com/cuda-downloads)

![选择LINUX->系统框架->WSL-Ubuntu->最新版本->deb(local)](images/wsl_cuda/CUDA.png)

```bash
# 根据英伟达CUDA下方提示命令复制执行
# 配置环境变量(要根据自己的CUDA版本修改)
vim ~/.bashrc
# 添加如下内容
export CUDA_HOME=/usr/local/cuda-12.6
export PATH=/usr/local/cuda-12.6/bin:$PATH 
export LD_LIBRARY_PATH=/usr/local/cuda-12.6/lib64:$LD_LIBRARY_PAT
# 环境生效
source ~/.bashrc
# 测试
nvcc -V
```

![配置成功](images/wsl_cuda/nvcc.png)

{% note success %}
配置完成
{% endnote %}


## CUDNN

[cudnn下载地址](https://developer.nvidia.com/cudnn-downloads)

{% tabs CUDNN %}
<!-- tab 通用安装方式 -->
![根据情况选择](images/wsl_cuda/cudnn.png)

```bash
# 根据英伟达CUDA下方提示命令复制执行
# [wget-name] 为下载的文件名例如：cudnn-linux-x86_64-9.6.0.74_cuda12-archive
# [version]为上一步安装的cuda版本例如：12.6

tar -xvf [wget-name].tar.xz
sudo cp cudnn-*-archive/include/cudnn*.h /usr/local/cuda-[version]/include
sudo cp -P cudnn-*-archive/lib/libcudnn* /usr/local/cuda-[version]/lib64 
sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda-[version]/lib64/libcudnn*

# 测试
cat /usr/local/cuda/include/cudnn_version.h | grep CUDNN_MAJOR -A 2
```
<!-- endtab -->
<!-- tab Ubuntu(20.04 to 24.04) -->

{% note info %}
此方法网上似乎见不到，但是配置简单，我测试没有问题，需要自行测试检验
{% endnote %}

[cudnn下载地址](https://developer.nvidia.com/cudnn-downloads)

![根据情况选择](images/wsl_cuda/cudnn2.png)

{% note red fa-bolt %}
只执行Installation Instructions:内指令即可后边安装cuda的指令不要执行
{% endnote %}

<!-- endtab -->
{% endtabs %}

{% note success  %}
配置完成
{% endnote %}


## TensorRT

[下载地址](https://developer.nvidia.com/tensorrt/download)

[官方文档](https://docs.nvidia.com/deeplearning/tensorrt/archives/index.html#trt_10)

![选择最新版本](images/wsl_cuda/TensorRT.png)

{% note info %}
选择合适的版本，我这里使用最新版本
{% endnote %}

![选择版本](images/wsl_cuda/RTSelect.jpg)

{% note info %}
同意许可->选择版本->确认cuda版本->选择自己的指令集和操作系统
{% endnote %}

{% tabs TensorRT %}

<!-- tab 通用安装方式 -->
![下载存档](images/wsl_cuda/RTSelectG.jpg)

```bash
# wget 下载地址修改为浏览器选择后右键复制的地址
wget https://developer.nvidia.com/downloads/compute/machine-learning/tensorrt/10.7.0/tars/TensorRT-10.7.0.23.Linux.x86_64-gnu.cuda-12.6.tar.gz
# 文件名为下载的文件名
tar -xzvf TensorRT-10.7.0.23.Linux.x86_64-gnu.cuda-12.6.tar.gz
# 转移到合适的文件夹
sudo mv TensorRT-10.7.0.23 /usr/local/TensorRT-10.7.0.23
# 配置环境变量
vim ~/.bashrc
# 添加 TensorRT 的绝对路径库目录添加到环境变量中 LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/usr/local/TensorRT-10.7.0.23/lib:$LD_LIBRARY_PATH
# 可选，添加 trtexec 到环境变量
export PATH=/usr/local/TensorRT-10.7.0.23/bin:$PATH
# 环境生效
source ~/.bashrc
```

** 配置项目环境 **
```bash
# 激活虚拟环境后执行安装
# cp3x 注意修改为自己的python版本
pip install /usr/local/TensorRT-10.7.0.23/python/tensorrt-*-cp3x-none-linux_x86_64.whl
# 可选，安装TensorRT精简和分发运行时环境的wheel文件
python3 -m pip install tensorrt_lean-*-cp3x-none-linux_x86_64.whl
python3 -m pip install tensorrt_dispatch-*-cp3x-none-linux_x86_64.whl
```

<!-- endtab -->
 
<!-- tab Ubuntu(20.04 to 24.04) -->
![下载存档](images/wsl_cuda/RTSelectU.jpg)

[官方文档](https://docs.nvidia.com/deeplearning/tensorrt/archives/tensorrt-1070/install-guide/index.html#installing-debian)

```bash
# wget 下载地址修改为浏览器选择后右键复制的地址
wget https://developer.nvidia.com/downloads/compute/machine-learning/tensorrt/10.7.0/local_repo/nv-tensorrt-local-repo-ubuntu2404-10.7.0-cuda-12.6_1.0-1_amd64.deb
# 文件名为下载的文件名
sudo dpkg -i nv-tensorrt-local-repo-ubuntu2404-10.7.0-cuda-12.6_1.0-1_amd64.deb
# 确认文件夹名称
sudo cp /var/nv-tensorrt-local-repo-ubuntu2404-10.7.0-cuda-12.6/*-keyring.gpg /usr/share/keyrings/
# 更新缓存
sudo apt-get update
# 针对完整的C++和Python运行时环境
sudo apt-get install tensorrt
# 如果只需精简运行时环境，而非tensorrt
sudo apt-get install libnvinfer-lean10
sudo apt-get install libnvinfer-vc-plugin10
# 针对精简运行时环境的Python包
sudo apt-get install python3-libnvinfer-lean
# 如果只需分发运行时环境，而非tensorrt
sudo apt-get install libnvinfer-dispatch10
sudo apt-get install libnvinfer-vc-plugin10
# 针对分发运行时环境的Python包
sudo apt-get install python3-libnvinfer-dispatch
# 对于所有不包含示例的TensorRT Python包
python3 -m pip install numpy
sudo apt-get install python3-libnvinfer-dev
# 如果只想为精简或分发运行时环境安装Python包，请单独指定这些包，而不是安装dev包。
# 如果您需要为非系统默认Python版本的Python模块，则应从tar包中直接安装*.whl文件。
```

<!-- endtab -->
{% endtabs %}

{% note success  %}
配置完成
{% endnote %}

## 环境测试

```python
import torch

# CUDA
device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
print(device)

x = torch.Tensor([2.1])
xx = x.cuda()
print(xx)

# CUDNN
from torch.backends import cudnn
print('cudann is ' + str(cudnn.is_acceptable(xx)))

# TensorRT
import tensorrt
print(tensorrt.__version__)
assert tensorrt.Builder(tensorrt.Logger())
# 如果卡住说明安装存在问题
```

![测试](images/wsl_cuda/test.jpg)
