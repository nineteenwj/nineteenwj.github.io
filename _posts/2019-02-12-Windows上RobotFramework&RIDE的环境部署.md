---
layout: post
title:  "Windows上RobotFramework&RIDE的环境部署"
date:   2019-02-12 22:34
categories: 不作恶
---

Robot Framework是一套在python环境中运行的以关键字驱动的自动化测试框架，RIDE则是基于Robot Framework开发的图形界面，包含Test Data编辑、测试用例创建、测试流程控制等功能。 

Robot Framework测试用例编写简单，并提供几十种library，可用于测试Web、Android、iOS、FTP、database等等，非常的方便快捷且易于学习。

# Step-by-step guide

## 1. 安装python2.7.12  
下载 pthon2.7.12 windows x86-64版本（[python-2.7.12.amd64.msi](https://www.python.org/ftp/python/2.7.12/python-2.7.12.amd64.msi)）并默认安装在C:\Python27\下。

**NOTE：robotframework本身是支持python3的，但图形化编辑工具RIDE只支持python2，因此本文中采用python2.7.12**

### 1.1 部署环境
右击“此电脑”，选择“属性”，在弹出面板中点击左边栏“高级系统属性”，在弹出的“系统属性”面板中选择“环境变量”，在“系统变量”的“Path”中添加路径 ***“C:\Python27”*** 和 ***“C:\Python27\Scripts”***

## 2.  安装wxPython  
wxPython 是python 的一个GUI库，由于RIDE 基于wxPython库开发，所以在开发RIDE之前必须先安装wxPython。

**a. 安装方式一 pip安装** 运行cmd.exe，进入python安装目录，运行命令：

```
pip.exe install wmPython  
```
**b. 安装方式二 安装包安装** 直接下载windows x86-64安装包（[wxPython2.8-win64-unicode-2.8.12.1-py27.exe](https://sourceforge.net/projects/wxpython/files/wxPython/2.8.12.1/wxPython2.8-win64-unicode-2.8.12.1-py27.exe/download)）并安装 

## 3. 安装Robot Framework
**a. 安装方式一 pip安装** 在cmd.exe中运行命令：
```
pip.exe install robotframework
```
**b. 安装方式二 源码安装** 下载 robotframework-3.0源码 （[Source code](https://files.pythonhosted.org/packages/93/15/35b58d06f35e2c12fc170c8d2aa46a8617beca52d240849388d9bf0f8386/robotframework-3.0.tar.gz)） 

 i. 解压该tar包（例如解压路径为D:\Tools\robotframework-3.0\robotframework-3.0）

 ii. 运行cmd.exe，进入robotframework-3.0的解压路径，运行命令：

```
python.exe setup.py install 
```

## 4. 安装RIDE
**a.  安装方式一 pip安装** 在cmd.exe中运行命令：
```
pip.exe install robotframework-ride
```
**b. 安装方式二 源码安装** 下载RIDE的源码 （[Source code](https://github.com/robotframework/RIDE/archive/v1.5.2.1.tar.gz)）

 i. 解压该 tar 包  

 ii. 运行cmd.exe，进入robotframework-ride的解压路径，运行命令：

```
C:\Python27\python.exe  setup.py install
```
**c. 安装方式三 安装包安装** 下载RIDE的安装程序 （[robotframework-ride-1.5.2.1.win-amd64.exe](https://github.com/robotframework/RIDE/releases/download/v1.5.2.1/robotframework-ride-1.5.2.1.win-amd64.exe)）

## 5. 出错处理
双击运行RIDE（C:\Python27\Scripts\ride.py），启动RIDE的窗口界面，在RIDE中导入任意测试用TestCase，Run Tests，TestCase的编码方式可能带来如下错误：
```python
UnicodeDecodeError: 'utf8' codec can't decode byte 0xb2 in position 12: invalid start byte
```
**解决方式：**
>i. 编辑 C:\Python27\Lib\site-packages\robotide\contrib\testrunner\testrunner.py，
>将 **line400:** return result.decode(**'UTF-8****'**) 改为 return  result.decode('**GBK**')
>
>ii. 重启RIDE，Run Tests，则可成功运行自动化测试


至此，RobotFramework&RIDE的自动化测试框架部署已全部完成。

# Related articles
- Robot Framework [官方网站](http://robotframework.org)
- Robot Framework [Github](https://github.com/robotframework/RIDE/wiki)