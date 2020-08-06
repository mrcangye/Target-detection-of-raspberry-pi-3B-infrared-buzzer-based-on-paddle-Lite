# 基于Paddle-Lite的树莓派3B红外蜂鸣目标检测

## 实验器材

本次使用的是老旧的**树莓派3B**以及**树莓派官方系统**，如果手头上没有树莓派但是想尝试的同学可以去购买新出的型号4版本，新的版本的性能更好。
用到的模块有**红外感应**模块，**蜂鸣器**，**树莓派摄像头**，**杜邦线**若干，**面包板**，面包板用的**排线**等。

## 参考文档

### Paddle-Lite GitHub

[https://github.com/PaddlePaddle/Paddle-Lite-Demo](https://github.com/PaddlePaddle/Paddle-Lite-Demo)

### Paddle-Lite Linux(ARM) Demo

[https://paddle-lite.readthedocs.io/zh/latest/demo_guides/linux_arm_demo.html](https://paddle-lite.readthedocs.io/zh/latest/demo_guides/linux_arm_demo.html)


## 思路以及来源

之前是想使用本地编译Paddle-Lite的，但是做到最后发现树莓派官方的系统少了个包进行不了交叉编译，囿于时间关系，暂时没有和这个问题死磕到底。如果后期能够解决会及时补充。因此，如果想在树莓派本地编译的话暂时不建议使用官方树莓派镜像，可以尝试使用基于linux的各类三方镜像。
例如ubuntu官方提供的镜像[https://ubuntu.com/download/raspberry-pi](https://ubuntu.com/download/raspberry-pi)，支持最新版的ubuntu20.04
因为以上原因，编译无法进行。但是[Paddle-Lite-Demo](https://github.com/PaddlePaddle/Paddle-Lite-Demo)倒是可以顺利运行，因此，后面的思路就是修改[Paddle-Lite-Demo](https://github.com/PaddlePaddle/Paddle-Lite-Demo)下的部分代码并结合Python语言实现我们需要的功能


## 树莓派前提配置

### 树莓派写入系统

首先要下载树莓派的写入工具以及镜像
分别对应Raspberry Pi Imager 与 Raspberry Pi Desktop (for PC and Mac)
镜像系统可以使用百度网盘的离线下载转存到百度网盘里，使用百度网盘会更快点
工具以及镜像下载链接
[https://www.raspberrypi.org/downloads/](https://www.raspberrypi.org/downloads/)

### 树莓派与电脑直连

这里需要一根网线连接树莓派与电脑。电脑端打开 控制面板\网络和 Internet\网络和共享中心
点击WLAN网络，点击属性
![](https://image.cangye.me/2020/08/06/基于paddle-lite的树莓派3b红外蜂鸣目标检测1.png)

进入属性后点击共享
![在这里插入图片描述](https://image.cangye.me/2020/08/06/基于paddle-lite的树莓派3b红外蜂鸣目标检测2.png)

将以上的选项打勾，确定即可。
接下来Win+R输入cmd打开终端
输入arp -a找到树莓派的IP地址
![在这里插入图片描述](https://image.cangye.me/2020/08/06/基于paddle-lite的树莓派3b红外蜂鸣目标检测3.png)

在这里我的树莓派IP地址是192.168.137.205
接下来下载putty这个ssh登录软件，输入ip地址连接即可
树莓派官方的用户名以及密码是
默认的用户名: pi
默认的密码: raspberry

### 开启摄像头以及VNC远程功能

使用putty进入树莓派终端输入
`sudo raspi-config` 
上下方向键移动，选择第5个 Interfacing Options进入
![在这里插入图片描述](https://image.cangye.me/2020/08/06/基于paddle-lite的树莓派3b红外蜂鸣目标检测4.png)

![在这里插入图片描述](https://image.cangye.me/2020/08/06/基于paddle-lite的树莓派3b红外蜂鸣目标检测5.png)

选择P1 Camera点击进入开启摄像机功能
同理，进入P3 VNC开启VNC功能
这样我们在自己的电脑也要下载VNC以便可以远程可视化连接树莓派。

### 引脚图

![在这里插入图片描述](https://image.cangye.me/2020/08/06/基于paddle-lite的树莓派3b红外蜂鸣目标检测6.png)

此处我们使用树莓派的物理引脚


### 蜂鸣器

![在这里插入图片描述](https://image.cangye.me/2020/08/06/基于paddle-lite的树莓派3b红外蜂鸣目标检测7.jpg)

这里使用的是低电平触发的蜂鸣器，引脚为三个如图示。这里我们VCC引脚接入树莓派3.3V 物理引脚GPIO 1号口，I/O引脚接入到物理引脚37号口，GND接地引脚接入树莓派物理引脚39号口

### 红外感应器

![在这里插入图片描述](https://image.cangye.me/2020/08/06/基于paddle-lite的树莓派3b红外蜂鸣目标检测8.jpg)

从图示的位置来看，红外感应器的引脚分别是VCC引脚，I/O引脚，GND引脚。
这里VCC引脚接入树莓派物理引脚2号口，I/O引脚接入到物理引脚32号口，GND接地引脚接入树莓派物理引脚34号口

### 摄像头

使用的是软排线的树莓派摄像头。

## [Paddle-Lite-Demo](https://github.com/PaddlePaddle/Paddle-Lite-Demo)的下载以及安装

我们首先要进入[Paddle-Lite-Demo](https://github.com/PaddlePaddle/Paddle-Lite-Demo)的项目页面查看文档，官方文档是最重要的。注意树莓派对应ARMLinux版本系统
我们看到：

> ARMLinux
>
> - RK3399（[Ubuntu 18.04](http://www.t-firefly.com/doc/download/page/id/3.html)） 或 树莓派3B（[Raspbian Buster with desktop](https://www.raspberrypi.org/downloads/raspbian/)），暂时验证了这两个软、硬件环境，其它平台用户可自行尝试；
> - 支持树莓派3B摄像头采集图像，具体参考[树莓派3B摄像头安装与测试](https://github.com/PaddlePaddle/Paddle-Lite-Demo/blob/master/PaddleLite-armlinux-demo/enable-camera-on-raspberry-pi.md)
> - gcc g++ opencv cmake的安装（以下所有命令均在设备上操作）
>
> $ sudo apt-get update
> $ sudo apt-get install gcc g++ make wget unzip libopencv-dev pkg-config
> $ wget [https://www.cmake.org/files/v3.10/cmake-3.10.3.tar.gz](https://www.cmake.org/files/v3.10/cmake-3.10.3.tar.gz)
> $ tar -zxvf cmake-3.10.3.tar.gz
> $ cd cmake-3.10.3
> $ ./configure
> $ make
> $ sudo make install

按文档教程安装相应软件包。由于是比较老旧的3B，所以编译耗时比较长。要耐心等待
完成后往下看：

> $ git clone [https://github.com/PaddlePaddle/Paddle-Lite-Demo](https://github.com/PaddlePaddle/Paddle-Lite-Demo)

按照文档说明下载克隆示例库，这里下载可能比较慢也要等待，也可以试试下载我在码云复制的版本：[https://gitee.com/mrcangye/Paddle-Lite-Demo](https://gitee.com/mrcangye/Paddle-Lite-Demo)
仓库下载好后继续看文档

> ARMLinux
>
> - 模型和预测库下载

> $ cd Paddle-Lite-Demo/PaddleLite-armlinux-demo

> $ ./download_models_and_libs.sh # 下载模型和预测库

> - 图像分类Demo的编译与运行（以下所有命令均在设备上操作）

> $ cd Paddle-Lite-Demo/PaddleLite-armlinux-demo/image_classification_demo

> $ ./run.sh armv8 # RK3399

> $ ./run.sh armv7hf # 树莓派3B

> 在终端打印预测结果和性能数据，同时在build目录中生成result.jpg。

> - 目标检测Demo的编译与运行（以下所有命令均在设备上操作）

> $ cd Paddle-Lite-Demo/PaddleLite-armlinux-demo/object_detection_demo

> $ ./run.sh armv8 # RK3399

> $ ./run.sh armv7hf # 树莓派3B

> 在终端打印预测结果和性能数据，同时在build目录中生成result.jpg。

按照文档操作即可。
到这里示例就安装完毕了。
在这次实验中，我用到的是目标检测的Demo，故我们要看看目标检测里的run.sh文件
run.sh文件位于Paddle-Lite-Demo/PaddleLite-armlinux-demo/object_detection_demo文件夹里
这里我们打开看看。由于我们使用的是树莓派3B，所以我们要把第4行使用#注释掉，把第5行的注释去掉。

```powershell
#!/bin/bash

# configure
#TARGET_ARCH_ABI=armv8 # for RK3399, set to default arch abi
TARGET_ARCH_ABI=armv7hf # for Raspberry Pi 3B
PADDLE_LITE_DIR=../Paddle-Lite
if [ "x$1" != "x" ]; then
    TARGET_ARCH_ABI=$1
fi

# build
rm -rf build
mkdir build
cd build
cmake -DPADDLE_LITE_DIR=${PADDLE_LITE_DIR} -DTARGET_ARCH_ABI=${TARGET_ARCH_ABI} ..
make

#run
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:${PADDLE_LITE_DIR}/libs/${TARGET_ARCH_ABI} ./object_detection_demo ../models/ssd_mobilenet_v1_pascalvoc_for_cpu/model.nb ../labels/pascalvoc_label_list ../images/new.jpg ../result.jpg

```

我们再看最后一行末尾两个jpg的代码。
这里的意思应该是输入图片在run.sh同级的image文件夹里，结果文件是在run.sh同级的文件夹下。这里我们把dog.jpg改成new.jpg。这个new.jpg作为树莓派摄像头拍照生成的图片名字。实验时候我们把拍照的图片名改成new.jpg并放在这里就可以了。

## python程序的编写

这里使用python对目标检测程序和树莓派各模块进行连通。
代码如下：

```python
import RPi.GPIO as GPIO
import time
import os
import sys

def init():
    '''引脚初始化'''
    GPIO.setwarnings(False)
    GPIO.setmode(GPIO.BOARD)
    GPIO.setup(32,GPIO.IN)
    GPIO.setup(37,GPIO.OUT)
    pass

def beep():
    '''蜂鸣器配置'''
    while GPIO.input(32):
        GPIO.output(37,GPIO.LOW)
        time.sleep(0.5)
        GPIO.output(37,GPIO.HIGH)
        time.sleep(0.5)

def dect():
    '''红外感应器配置'''
    for i in range(1,101):
        if GPIO.input(32) == True:
            print(time.strftime('%Y-%m-%d %H:%M:%S',time.localtime(time.time()))+" Someone is coming !")
            beep()
            #树莓派拍照，并将照片放在目标检测的对应目录里
            os.system("raspistill -o /home/pi/Paddle-Lite-Demo/PaddleLite-armlinux-demo/object_detection_demo/images/new.jpg -w 640 -h 480")
            #调用目标检测程序
            os.system("cd /home/pi/Paddle-Lite-Demo/PaddleLite-armlinux-demo/object_detection_demo && sh run.sh")
        else:
            GPIO.output(37,GPIO.HIGH)
            print("Nobody!")
        time.sleep(2)

time.sleep(0.5)
init()
dect()
GPIO.cleanup()


```

## 效果展示

![在这里插入图片描述](https://image.cangye.me/2020/08/06/基于paddle-lite的树莓派3b红外蜂鸣目标检测9.jpg)
![在这里插入图片描述](https://image.cangye.me/2020/08/06/基于paddle-lite的树莓派3b红外蜂鸣目标检测10.jpg)
![在这里插入图片描述](https://image.cangye.me/2020/08/06/基于paddle-lite的树莓派3b红外蜂鸣目标检测11.png)
![在这里插入图片描述](https://image.cangye.me/2020/08/06/基于paddle-lite的树莓派3b红外蜂鸣目标检测12.jpg)
![在这里插入图片描述](https://image.cangye.me/2020/08/06/基于paddle-lite的树莓派3b红外蜂鸣目标检测13.jpg)

## 后记

本次的实验尽管较为顺利完成，但是囿于材料，技术等原因，尚且有许多不足：摄像机的拍照时间稍微有点长，没有云台功能导致摄像机不能跟随目标移动，程序的响应速度尚不能跟上目前市面上的摄像机，硬件性能不足等。感兴趣的同学可以尝试复现并进一步完善这次的实验。
