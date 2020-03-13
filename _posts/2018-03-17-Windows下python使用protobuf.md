---
layout: post
title:  "Windows下python使用protobuf"
date:   2018-03-17 10:12
categories: 不作恶
---

本文主要介绍如何在windows上安装protobuf，以及如何在python下使用protobuf。

# What is protocol buffers

首先介绍一下什么是protobuf， protobuf 是由google开发完成并放到github上分享的一套打包数据的机制，官方介绍如下：

> Protocol buffers are a flexible, efficient, automated mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages. You can even update your data structure without breaking deployed programs that are compiled against the "old" format.

简单来说Protobuf 是一种比XML更优秀的序列化数据结构的机制，google在其自身的RPC通信中全面应用protobuf来对传输数据进行收发。 发送端发送数据之前将数据序列化打包，再通过各种通信方式传输到接收端，由接收端反序列化从而获得原始数据。 可实现跨平台、跨语言的数据封装与解析，比XML更简单，更小巧，效率更高。 想要更详细地了解google自卖自夸请直达官网链接[https://developers.google.com/protocol-buffers/docs/overview](https://developers.google.com/protocol-buffers/docs/overview)，当然protobuf本身也是个好东西，不然google也不会自己全面用它。

## 1. 下载protobuf  python 3.0  
下载链接：[轻戳这里](https://github.com/google/protobuf/releases/tag/v3.0.0)  
下载两个包：[protobuf-python-3.0.0.zip](https://github.com/google/protobuf/releases/download/v3.0.0/protobuf-python-3.0.0.zip) 以及 [protoc-3.0.0-win32.zip](https://github.com/google/protobuf/releases/download/v3.0.0/protoc-3.0.0-win32.zip)  
**protobuf-python-3.0.0**为protobuf的安装包，**protoc-3.0.0-win32**包含protobuf的编译器protoc的win32版本，用以编译*.proto文件。

## 2. 安装protobuf  
### a. 确认版本  
使用protobuf的python版本必须在2.6以上，本文采用**python2.7.12**，protoc的版本要与protobuf的版本保持一致。 
在cmd中运行 
```shell
python.exe -V 
python 2.7.12 
protoc.exe --version 
libprotoc 3.0.0
```
### b. 安装setuptools 
如果python2.7的版本大于2.7.9，在安装python时setuptools已自动安装，否则则需要手动下载[setuptools](https://packaging.python.org/installing/)

### c. Build 
进入目录***protobuf-3.0.0\python\\***，运行以下命令: 
```shell
python.exe setup.py build 
```
![img](/assets/posts/Windows下python使用protobuf/protobufbuild)

将**protoc-3.0.0-win32\bin\protoc.exe** 复制到***protobuf-3.0.0\src***，
重新运行 ，生成大量*.py 
```shell
python.exe setup.py build
```

### d. Test  
测试protobuf是否build成功： 
```shell
python.exe setup.py test 
```
![img](/assets/posts/Windows下python使用protobuf/testprotobuf)

### e. install  

运行如下命令，实际上build那步已经安装得差不多了 
```
python.exe setup.py install 
```
![img](/assets/posts/Windows下python使用protobuf/protobufinstall)

安装six时报了个error的错误，下载失败，暂时不去理会。

## 3. 测试protobuf  
### a. 生成py文件  
创建一个proto文件test.proto，这实际上是定义数据类型，类似c中的struct

```
message CDevice
{
   optional int32 devId = 1;
   optional string name = 2;
}
```

该数据类型名为CDevice，其中包含两个属性，一个是int32型的设备Id devId，一个是string型的设备名称 name 运行命令，生成test_pb2.py，该py文件需要import到测试程序中。
如果proto文件中没有指定生成prot2还是proto3，默认生成proto2：
```
protoc -I=./ --python_out=./ test.proto 
```
![img](/assets/posts/Windows下python使用protobuf/protocgen) 

**-I** 为proto文件的路径，**--python_out=./** 表示在当前路径下由指定的\*.proto生成python可用的\*.py文件

### b. 编写测试程序
```python
#!/usr/bin/env python 
import test_pb2 
import traceback 
import sys

try:
	sendData = test_pb2.CDevice()      
	sendData.devId = 9      
	sendData.name = 'USB'
	
	#Serialize      
	sendDataStr = sendData.SerializeToString()      
	#print serialized string value      
	print 'serialized string:', sendDataStr      
	#------------------------#      
	#  message transmission  #      
	#------------------------#      
	receiveDataStr = sendDataStr      
	receiveData = test_pb2.CDevice()

	#Deserialize      
	receiveData.ParseFromString(receiveDataStr)      
	print 'pares serialize string, return: devId = ', \
	receiveData.devId, ', name = ', receiveData.name 
	
except Exception, e:      
	print Exception, ':', e      
	print traceback.print_exc()      
	errInfo = sys.exc_info()      
	print errInfo[0], ':', errInfo[1] 
```
注：在message.py中可以查到protobuf的调用方法，主要有序列化和反序列化。中间省略了数据传输部分，数据序列化后可以用rpc、socket或者其他方式传输。

运行后发现没有安装descriptor.py里依赖的module six。Six实际上是一个用以适配 Python2 和 Python3的一个模块，也就是说，安装Six后，其他模块可以在不更改代码也能同时支持 Python2 和 Python3 。之所以取名 "six"，是因为 2*3=6。

Six的官方解释 [轻戳此处](https://six.readthedocs.io/)

> Six provides simple utilities for wrapping over differences between Python 2 and Python 3. It is intended to support codebases that work on both Python 2 and 3 without modification. six consists of only one Python file, so it is painless to copy into a project. 

回想之前安装protobuf时的error，就是指下载这个six出错。 手动安装six，运行如下命令，这次倒很顺利地安装成功。

```shell
pip.exe install six
```
![img](/assets/posts/Windows下python使用protobuf/installsix)

重新运行测试程序后，输出如下结果

![img](/assets/posts/Windows下python使用protobuf/testresult)

序列化后的string是不可读的，反序列化后得到之前sendData的赋值。

# 相关链接

1. 编程指导：[https://developers.google.com/protocol-buffers/docs/tutorials](https://developers.google.com/protocol-buffers/docs/tutorials)
2. protocol buffer 编码方式：[https://developers.google.com/protocol-buffers/docs/encoding](https://developers.google.com/protocol-buffers/docs/encoding)
3. 10多种语言API参考文档：[https://developers.google.com/protocol-buffers/docs/reference/overview](https://developers.google.com/protocol-buffers/docs/reference/overview)
4. protobuf 语法：[https://developers.google.com/protocol-buffers/docs/proto](https://developers.google.com/protocol-buffers/docs/proto)
5. 样式指南： [https://developers.google.com/protocol-buffers/docs/style](https://developers.google.com/protocol-buffers/docs/style)
6. 官方示例：[https://developers.google.com/protocol-buffers/docs/pythontutorial](https://developers.google.com/protocol-buffers/docs/pythontutorial)

