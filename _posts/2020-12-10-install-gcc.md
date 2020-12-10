---
layout: post
title: ubuntu安装低版本gcc
tags: clang/llvm
excerpt: 如何安装另外的gcc，如何指定默认的版本
---  

### Ubuntu更新下载源    

由于llvm2.6的代码实在有些年头了，我在mac上编译会提示一些模板的问题，以及编译前向依赖问题，所以
我打算还是在linux上部署，经过网上搜索，ubuntu 14.04 64位，gcc 4.4可以构建llvm2.6。于是昨天我购买了
阿里云共享型服务器c1m4(1 cpu, 4G memory)，盘40G，后面打算IDE通过SSH remote来远程连接它。
ubuntu 14.04自带gcc 4.8，我需要安装低版本gcc，apt-get的时候搜索不到源，所以需要先添加下ubuntu的
下载源。[参考链接](https://www.cnblogs.com/mengw/p/11408118.html). 步骤如下。

``` 
vim /etc/apt/sources.list  

#阿里云
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse 

然后退出保存执行

sudo apt-get update    
``` 

### 安装gcc4.4配置默认的优先级

``` 
#下载
sudo apt-get install gcc-4.4 gcc-4.4-multilib g++-4.4 g++-4.4-multilib 

#配置级别   
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.4 100
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 80

sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.4 100
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.8 80   

#查询配置  
update-alternatives --query gcc
update-alternatives --query g++   

#最后命令行验证
gcc -v 
```   

### 参考链接   

https://zhuanlan.zhihu.com/p/24141227    

https://www.cnblogs.com/mengw/p/11408118.html   

https://www.cnblogs.com/xlqtlhx/p/8012249.html   

https://huanghailiang.github.io/2017/07/22/ubuntu-Reduce-the-version-gcc-g++/   