---
layout: post
title: llvm的编译和使用
tags: clang/llvm
excerpt: 终于能使用llvm了
---  

### llvm的编译   

参考官网[链接](http://llvm.org/docs/CMake.html#embedding-llvm-in-your-project)即可，下载好llvm对应版本，执行
如下的命令.

```  
mkdir build && cd build

# -DLLVM_TARGETS_TO_BUILD=X86
cmake ../llvm-3.5xxx -DCMAKE_BUILD_TYPE=Debug

cmake --build .  

#指令一个安装目录
cmake -DCMAKE_INSTALL_PREFIX=/tmp/llvm -P cmake_install.cmake 

在bashrc里面设置好环境PATH能搜索到安装的目录. 

eg: $PATH=/xxx/llvm-install/:/xxx/llvm-install/bin:$PATH;

```  

### llvm使用

由于配置好了环境path，所以可以很方便的使用cmake和llvm-config命令了.

如果使用cmake的find-package功能，参考官网的例子, 如下。

``` 
cmake_minimum_required(VERSION 3.13.4)
project(SimpleProject)

find_package(LLVM REQUIRED CONFIG)

message(STATUS "Found LLVM ${LLVM_PACKAGE_VERSION}")
message(STATUS "Using LLVMConfig.cmake in: ${LLVM_DIR}")

# Set your project compile flags.
# E.g. if using the C++ header files
# you will need to enable C++11 support
# for your compiler.

include_directories(${LLVM_INCLUDE_DIRS})
add_definitions(${LLVM_DEFINITIONS})

# Now build our tools
add_executable(simple-tool tool.cpp)

# Find the libraries that correspond to the LLVM components
# that we wish to use
llvm_map_components_to_libnames(llvm_libs support core irreader)

# Link against LLVM libraries
target_link_libraries(simple-tool ${llvm_libs})  
``` 

如果使用llvm-config命令，可以参考如下。   

``` 
LLVM_CONFIG = llvm-config
LLVM_CXXFLAGS += $(shell $(LLVM_CONFIG) --cxxflags)
LLVM_LDFLAGS := $(shell $(LLVM_CONFIG) --ldflags)
LLVM_LIBS = $(shell $(LLVM_CONFIG) --libs bitwriter core support)
llvm_model_src = ModuleMaker.cpp
test_model:
c++ $(llvm_model_src) $(LLVM_CXXFLAGS) $(LLVM_LIBS) $(LLVM_LDFLAGS) -lpthread -ldl -o ModuleMaker  
``` 

llvm-config可以很方便的提供cflags和dflags.
