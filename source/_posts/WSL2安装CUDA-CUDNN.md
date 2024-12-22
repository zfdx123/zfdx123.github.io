---
layout: post
title: WSL2 环境下配置CUDA和CUDNN
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
sudo vim ~/.bashrc
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

配置成功


## CUDNN

[cudnn下载地址](https://developer.nvidia.com/cudnn-downloads)

![根据情况选择](images/wsl_cuda/cudnn.png)

```bash
# 根据英伟达CUDA下方提示命令复制执行
# [wget-name] 为下载的文件名例如：cudnn-linux-x86_64-9.6.0.74_cuda12-archive.tar.xz
# [version]为上一步安装的cuda版本例如：12.6

tar -xvf [wget-name].tar.xz
sudo cp cudnn-*-archive/include/cudnn*.h /usr/local/cuda-[version]/include
sudo cp -P cudnn-*-archive/lib/libcudnn* /usr/local/cuda-[version]/lib64 
sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda-[version]/lib64/libcudnn*

# 测试
cat /usr/local/cuda/include/cudnn_version.h | grep CUDNN_MAJOR -A 2
```

配置成功

## CUDNN (配置方式二)

{% note info %}
此方法网上似乎见不到，但是配置简单，我测试没有问题，需要自行测试检验
{% endnote %}

[cudnn下载地址](https://developer.nvidia.com/cudnn-downloads)

![根据情况选择](images/wsl_cuda/cudnn2.png)

{% note red fa-bolt %}
只执行Installation Instructions:内指令即可后边安装cuda的指令不要执行
{% endnote %}

## 环境测试

```python
import torch

device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
print(device)

x = torch.Tensor([2.1])
xx = x.cuda()
print(xx)

# CUDNN TEST
from torch.backends import cudnn
print('cudann is ' + str(cudnn.is_acceptable(xx)))

```