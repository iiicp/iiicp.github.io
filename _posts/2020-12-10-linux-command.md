---
layout: post
title: 常用linux命令  
tags: clang/llvm
excerpt: 一些常用命令 
---   

### 查看系统版本 

```
uname -a 
uname -r
lsb_release -a
``` 

### 修改主机名 

```
临时修改主机名 
hostname 新主机名

永久修改主机名
vim /etc/hostname 
reboot 
```

### 查看目录大小 

``` 
du -sh 目录名
df -hl 目录名
```  
 

### 解压命令  

```  
tar -zxvf filename.tar.gz
tar -jxvf filename.tar.xz
tar -Zxvf filename.tar.Z 
```  

### 文件查找命令   

``` 
find / -name xxx 2>/dev/null      
``` 

### 进程/线程相关    

``` 
ps -ef | grep pulseaudio       
``` 

### elf格式相关    

```  
# 查看so库的搜索路径
readelf -d  xxxx.so  

objdump -tT xxx.so | grep "xxxx"
```  

### add2line  

``` 
# 0029a068表示crash的堆栈的地址，必须要编译so库的ndk版本   
./arm-linux-androideabi-4.9/prebuilt/darwin-x86_64/bin/arm-linux-androideabi-addr2line 0029a068 -C -f -e <so path>

android进程崩溃，crash日志会生成在/data/tombstones/
``` 