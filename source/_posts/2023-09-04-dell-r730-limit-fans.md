---
title: "dell R730 风扇转速100%"
date:   2023-09-04  17:00:00 +0800
layout: post
tag:
- OS
categories:
- Linux
---

dell R730 风扇转速100%

------

## 背景
dell r730 服务器安装的两块A2显卡，非dell官方指定的显卡，bios无法获取温度进行主动调速，所以把风扇转速自动调整为100%，部分情况下cpu/gpu占用率低下，但是风扇全功率运行，声音一言难尽。

## 解决方案

### 使用ipmitool手动调速

ipmitool安装：

```
apt install ipmitool
```

github地址：https://github.com/ipmitool/ipmitool

windows下载地址：https://www.dell.com/support/home/zh-cn/drivers/driversdetails?driverid=m63f3

### 调速

首先要关闭风扇自动调速功能，否则我们手动设置的转速是不会生效的。最后的0x00表示关闭自动调速，0x01表示开启自动调速。

```
ipmitool -I lanplus -H 192.168.3.250 -U root -P 1qaz@WSX raw 0x30 0x30 0x01 0x00
```

关闭自动调速之后，我们就可以按照我们自己的意愿来调整转速了，我这边设置为50%。

```
ipmitool -I lanplus -H 192.168.3.250 -U root -P 1qaz@WSX raw 0x30 0x30 0x02 0xff 0x32
```

最后的参数0x32表示转速的百分比的十六进制，0a表示10%，0f表示15%，32表示50%。

![DELL BMC ](/img/20230904-01.png)

通过调整发现，转速确实低了，之前一直稳定在20%-25%（5000+转）左右，功耗大概在170w。通过调低风扇转速，不仅静音了，还降低了功耗。

需要注意，如果在服务器本地执行的话，不用指定-H参数。

PS：
1、不是永久设置，服务器关闭电源，再次插上电源则需要重新调整。

2、如果是跑应用，为了防止烧显卡，建议直接100%

3、配置一下邮件预警，当温度到达一定程度，主动发送邮件，或者放开调速

## 优化
### 自动调速
#### 1、获取CPU温度
使用sensors工具获取CPU温度，安装命令如下：
```
sudo apt install lm-sensors
```

获取CPU温度命令行如下：
```
sensors 
```
也可以直接使用psutil库
pip install psutil
获取CPU温度，代码参考：
```
import psutil

def get_cpu_temperature():
    cpu_temperature = psutil.sensors_temperatures()['coretemp'][0].current
    return cpu_temperature

print("CPU Temperature:", get_cpu_temperature(), "°C")
```

#### 2、获取GPU温度
这个使用nvidia-smi获取，然后处理一下即可：
```
nvidia-smi -q -d TEMPERATURE | grep 'GPU Current Temp'| awk '{print $5}'
```
也可以使用GPUtil库来处理，安装命令：pip install GPUtil

参考代码：
```
import GPUtil

def get_gpu_temperature():
    gpus = GPUtil.getGPUs()
    gpu_temperature = gpus[0].temperature
    return gpu_temperature

print("GPU Temperature:", get_gpu_temperature(), "°C")
```
#### 自动调速和高温邮件预警
自动调速和高温邮件预警之类的可以根据以上内容，自行开发了，根据CPU和GPU温度实时调速，一定注意，GPU和CPU温度需要同时进行判断，由于服务器GPU上无风扇，只能被动散热，如果调整不当，容易烧显卡。

## 错误：

### Error: Unable to establish IPMI v2 / RMCP+ session

#### 原因
1、用户名或者密码错误

2、未启用ipml

#### 解决

1、设置密码：

```
Menu Overview -> IDRAC SETTINGS -> User Authentication
-> Click on the userID of your admin account -> Next
-> check "change your password" checkbox and enter the same (or new) password
-> Apply
```

2、启动ipml
![DELL BMC ](/img/20230904-02.png)
测试命令：

```
ipmitool -I lanplus -H 192.168.3.250 -U root -P 1qaz@WSX power status
```

![ipml test ](/img/20230904-03.png)

## 参考
- [https://stackoverflow.com/questions/51948745/error-unable-to-establish-ipmi-v2-rmcp-session](https://stackoverflow.com/questions/51948745/error-unable-to-establish-ipmi-v2-rmcp-session)
