---
layout: post
title: Windows PC连接Android adb shell的方法
author: Mike
keyword: Android
excerpt: 
postid: 2
---
今天突发奇想，想用跟笔记本连接的键鼠操作自己的安卓设备（具体操作先挖个坑之后再填）。
首先遇到的问题就是，如何用Windows PC连接自己的安卓设备，因为笔者的笔记本的USB接口有限，又有多部安卓设备，还想同时对多部Android设备进行调试，以下是探索到方法：

## 一、查看Windows的环境变量：
1. `win徽标键+R`         打开运行框
2. 键入`cmd`       打开命令指示符
3. `echo %PATH%`   获取当前的环境变量（误删环境变量/误删PATH时可以依照此方法获取环境变量）
        环境变量里有`C:\Android\;`表明已经安装ADB（Android Debug Bridge）

## 二、获取PC的公钥和私钥（RSA）：
- 在cmd命令指示符里执行`adb keygen C:\Android\.android\adbkey`
- 在`C:\Android\.android\`建立了两个文件：`adbkey`（私钥）和`adbkey.pub`（公钥）

## 二、（延伸学习）Android设备端的公钥和私钥：
Android设备端储存PC公钥的地址为：`/data/misc/adb/adb_keys`（需要root访问权限）#TODO 挖个坑之后来补

## 二、（延伸学习）公、私钥工作的原理：
当你执行“adb shell”时，adb.exe会将当前PC的公钥(或者公钥的hash值)(fingerprint)发送给android设备；这时，如果android上已经保存了这台PC的公钥，则匹配出对应的公钥进行认证，建立adb连接；如果android上没有保存这台PC的公钥，则会弹出提示框，让你确认是否允许这台机器进行adb连接，当你点击了允许授权之后，android就会保存了这台PC的`adbkey.pub`(公钥)；

## 三、在Android设备中开启USB调试
1. "设置 - 关于手机 - 版本号" 连按，直到系统弹出"进入开发者模式"的提示
2. "设置 - 开发者选项 - USB调试" 开启

## 四、连接adb实例：
###（方法一）USB有线连接：
1. 用USB将PC与Android设备连接；
2. cmd命令指示符执行`adb devices`
```cmd    
REM OUTPUT:
REM     [serialNumber] [state]
REM         [state]: offline/device
```
3. `adb shell`
```cmd
REM 错误信息一
adb server is out of date.  killing...
ADB server didn't ACK
* failed to start daemon *
```
```
REM 错误信息一 的 解决方法一：
REM     用管理员模式运行cmd
```
```
@echo off
REM 错误信息一 的 解决方法二：
tasklist /FI "IMAGENAME eq adb.exe"
REM     get the pid of adb.exe METHOD 1
REM or
    tasklist | findstr "adb"
REM     get the pid of adb.exe METHOD 2

REM [Optional]: 
REM     tasklist /?                         
REM         cmd查询参数帮助的方法，BTW cmd的class attribute帮助文档做的很清晰，值得参考与借鉴

taskkill /f /pid <pid_of_adb.exe>
REM     kill a task of pid
REM or
    taskkill /f /im "adb.exe"
REM     kill a task by image_name

adb kill-server
adb start-server
pause>nul
```
###（方法二）局域网内无线调试：
1. 在使用（方法一）有线连接到`adb devices`的时候执行`adb tcpip 5555`再断开连接
（步骤1的其他方式：若Android设备已经获取Root权限，也可以在Android设备中安装"终端模拟器(Terminal)"，并且在"终端模拟器"中依次执行以下命令：）
`setprop service.adb.tcp.port 5555`
`stop adbd`
`start adbd`
2. 在Android设备已连接WiFi的详情找到Android设备的IP地址，例如`192.168.1.100`
3. 管理员模式运行cmd/PowerShell，执行指令`adb connect 192.168.1.100:5555`
这里的端口号5555跟第一步设置为相同即可
4. `adb shell`

