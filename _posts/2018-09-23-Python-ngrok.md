---
layout: post
title: Python socket借助ngrok建立外网TCP连接
author: Mike
keyword: Python
tags: #python #内网穿透 #端口映射 #port forwarding #socket #TCP/IP #网络编程
postid: 4
---
目的是利用python3的socket库，建立以下两个终端之间的连接并传输简单的binary数据：

## 〇、设备简介

#### 服务器：
**设备**：Windows 10 PC

**终端**：cmd

**网络**：公网动态IP（由无线路由器接入）

#### 客户端：
**设备**：Android Phone

**终端**：[QPython3](https://www.coolapk.com/apk/com.hipipal.qpy3) - 终端

**网络**：移动4G


## 一、ngrok的配置：
- 点击这个链接-> [ngrok - secure introspectable tunnels to localhost](https://dashboard.ngrok.com/get-started) 
找到对应的版本，下载并注册
- 将`ngrok.exe`文件解压到喜欢的位置
- 将`ngrok.exe`所在的目录添加到Windows的环境变量里（Cortana搜索 “env” -> 编辑账户的环境变量 -> path -> 新建）
- 点击这个链接-> [ngrok auth](https://dashboard.ngrok.com/auth)
复制你的Authtoken
- [cmd] `ngrok authtoken [你的Authtoken]`

## 二、执行服务器的代码：
```python3
import socket
import threading
import time
# 源代码来自廖雪峰的python3教程
# https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432004374523e495f640612f4b08975398796939ec3c000#0

# 以下是每次TCP连接将要执行的线程
def tcplink(sock, addr):
    print('Accept new connection from %s:%s...' % addr)
    sock.send(b'Greating from Windows 10 PC server!')
    while True:
        data = sock.recv(1024)      # For this session, pause here, once client socket send message proceed following:
        time.sleep(1)
        if not data or data.decode('utf-8') == 'exit':
            break
        sock.send(('Hello, %s!' % data.decode('utf-8')).encode('utf-8'))
    sock.close()
    print('Connection from %s:%s closed.' % addr)

if __name__ == '__main__':
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 监听Localhost端口
    s.bind(('127.0.0.1', 9999))
    s.listen(5)
    print('Waiting for connection...')
    while True:
        sock, addr = s.accept()     # Paused here, once received connection proceed the following:
        t = threading.Thread(target=tcplink, args=(sock, addr))
        t.start()
```
## 三、使用ngrok开启服务器的TCP端口转发
- [cmd] `ngrok tcp 9999`
后边的数字与第二步服务器端代码中的端口号相同即可
- 在ngrok的运行信息里找到转发的公网地址和端口：如下图红线部分
![地址：0.tcp.ngrok.io 端口：19240](https://upload-images.jianshu.io/upload_images/12350396-3b32489170b7d4d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 四、执行客户端的代码
在安卓 QPython3* 中打开终端，并执行以下代码
将`ip`和`port`变量修改成第三步中ngrok提示的地址与端口
[*\* QPython3 下载地址*](https://www.coolapk.com/apk/com.hipipal.qpy3) 

```python3
import socket
import androidhelper

droid = androidhelper.Android()  # 处理弹出提示

ip = "0.tcp.ngrok.io"
port = 19354

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((ip, port))

datarecv = s.recv(1024).decode('utf-8')
droid.makeToast(datarecv) # 手机端将弹出 

# 发送数据
s.send(b'Steve')
# 接收数据
datarecv = s.recv(1024).decode('utf-8')
# 展示数据
droid.makeToast(datarecv)
```