# ADB-About
Android Debug Bridge Online document

---------

## 常用快速查看：

adb shell df 查看系统盘符

adb shell pm list packages -f 输出所有已经安装的应用

adb shell ls -i /system/lib | find "xxx" 查找是否包含xxx

adb shell dumpsys activity | findstr "mFocusedActivity" 通过adb查看当前显示的activity，其中在Windows下使用findstr，在Linux下使用grep

adb shell dumpsys activity | find "mF" 相当于过滤，只找名为"mF"的activity使用情况信息

adb shell dumpsys window windows | find "mCurrent"     获得当前活动窗口的信息，包名以及活动窗体

adb shell pm path com.migu.lobby 包名管理命令，获得对应包名的对应apk路径：

adb shell am set-debug-app -w <package_name> 设置调试应用，此设置是一次性的

adb shell am set-debug-app -w --persistent <package_name> 设置调试应用，使其一直处于调试模式，每次启动就会弹出 Waiting For Debugger 的窗口等待

adb shell am clear-debug-app 清理被标记为 debug 的 App

------

## ADB架构
ADB是一个C/S架构的应用程序，由三部分组成：
1. 运行在pc端的adb client：<br>
命令行程序”adb”用于从shell或脚本中运行adb命令。首先，“adb”程序尝试定位主机上的ADB服务器，如果找不到ADB服务器，“adb”程序自动启动一个ADB服务器。接下来，当设备的adbd和pc端的adb server建立连接后，adb client就可以向ADB servcer发送服务请求；
2. 运行在pc端的adb server：<br>
ADB Server是运行在主机上的一个后台进程。它的作用在于检测USB端口感知设备的连接和拔除，以及模拟器实例的启动或停止，ADB Server还需要将adb client的请求通过usb或者tcp的方式发送到对应的adbd上；
3. 运行在设备端的常驻进程adb demon (adbd)：<br>
程序“adbd”作为一个后台进程在Android设备或模拟器系统中运行。它的作用是连接ADB服务器，并且为运行在主机上的客户端提供一些服务；

------

## 正文

<span id="目录">

+ [一、基本用法](#基本用法)
  + [1.目标设备命令](#目标设备命令)
  + [2.启动和停止](#启动和停止)
  + [3.查看adb版本](#查看adb版本)
  + [4.获取root权限](#获取root权限)
  + [5.指定网络端口](#指定网络端口)
+ [二、设备相关](#设备相关)
  + [1.查询已连接设备](#查询已连接设备)
  + [2.设备型号](#设备型号)
  + [3.电池状况](#电池状况)
  + [4.屏幕分辨率](#屏幕分辨率)
  + [5.屏幕密度](#屏幕密度)
  + [6.显示屏参数](#显示屏参数)
  + [7.获取设备的android_id](#获取设备的android_id)
  + [8.IMEI](#IMEI)
  + [9.系统版本](#系统版本)
  + [10.获取IP地址](#获取IP地址)
  + [11.查看网络统计信息](#查看网络统计信息)
  + [12.管理网络连接](#管理网络连接)
  + [13.Mac地址](#Mac地址)
  + [14.CPU信息](#CPU信息)
  + [15.内存信息](#内存信息)
  + [16.查看Android设备系统属性](#查看Android设备系统属性)
+ [三、修改设置](#修改设置)
  + [1.分辨率](#分辨率)
  + [2.屏幕密度](#屏幕密度)
  + [3.显示区域](#显示区域)
  + [4.关闭USB调试模式](#关闭USB调试模式)
  + [5.允许/禁止访问非SDK-API](#允许/禁止访问非SDK-API)
  + [6.状态栏和导航栏的显示隐藏](#状态栏和导航栏的显示隐藏)
+ [四、应用管理](#应用管理)
  + [1.查看应用列表](#查看应用列表)
  + [2.按包名关键字查找应用](#按包名关键字查找应用)
  + [3.安装APK](#安装APK)
  + [4.卸载应用](#卸载应用)
  + [5.清除应用数据与缓存](#清除应用数据与缓存)
  + [6.查看前台Activity](#查看前台Activity)
  + [7.查看正在运行的Services](#查看正在运行的Services)
  + [8.查看应用详细信息](#查看应用详细信息)
  + [9.查看应用安装路径](#查看应用安装路径)
  + [10.与应用交互](#与应用交互)
  + [11.停止Service](#停止Service)
  + [12.发送广播](#发送广播)
  + [13.强制停止应用](#强制停止应用)
  + [14.收紧内存](#收紧内存)
+ [五、文件管理](#文件管理)
  + [1.复制设备里的文件到电脑](#复制设备里的文件到电脑)
  + [2.复制电脑里的文件到设备](#复制电脑里的文件到设备)
+ [六、实用功能](#实用功能)
  + [1.屏幕截图](#屏幕截图)
  + [2.录制屏幕](#录制屏幕)
  + [3.重新挂载system分区为可写](#重新挂载system分区为可写)
  + [4.查看连接过的WiFi密码](#查看连接过的WiFi密码)
  + [5.设置系统日期和时间](#设置系统日期和时间)
  + [6.重启手机](#重启手机)
  + [7.检测设备是否已root](#检测设备是否已root)
  + [8.使用Monkey进行压力测试](#使用Monkey进行压力测试)
  + [9.开启/关闭WiFi](#开启/关闭WiFi)
+ [七、模拟按键和输入](#模拟按键和输入)
  + [1.电源键](#电源键)
  + [2.菜单键](#菜单键)
  + [3.HOME键](#HOME键)
  + [4.返回键](#返回键)
  + [5.音量控制](#音量控制)
  + [6.媒体控制](#媒体控制)
  + [7.点亮/熄灭屏幕](#点亮/熄灭屏幕)
  + [8.滑动解锁](#滑动解锁)
  + [9.输入文本](#输入文本)
+ [八、查看日志](#查看日志)
  + [1.Android日志](#Android日志)
  + [2.示例1：按优先级过滤](#示例1：按优先级过滤)
  + [3.示例2：按tag和级别过滤日志](#示例2：按tag和级别过滤日志)
  + [4.示例3：显示某一TAG的日志信息](#示例3：显示某一TAG的日志信息)
  + [5.示例4：清理已经存在的日志](#示例4：清理已经存在的日志)
  + [6.示例5：将日志显示在控制台后退出](#示例5：将日志显示在控制台后退出)
  + [7.logcat格式化输出](#logcat格式化输出)
  + [8.内核日志](#内核日志)
+ [九、刷机相关命令](#刷机相关命令)
  + [1.重启到Recovery模式](#重启到Recovery模式)
  + [2.从Recovery重启到Android](#从Recovery重启到Android)
  + [3.重启到Fastboot模式](#重启到Fastboot模式)
  + [4.通过sideload更新系统](#通过sideload更新系统)
+ [十、安全相关命令](#安全相关命令)
  + [1.启用/禁用SELinux](#启用/禁用SELinux)
  + [2.启用/禁用dm_verity](#启用/禁用dm_verity)
+ [十一、adb shell](#adb-shell)
  + [1.查看进程](#查看进程)
  + [2.查看实时资源占用情况](#查看实时资源占用情况)
  + [3.显示activity相关的信息](#显示activity相关的信息)
  + [4.显示状态栏相关的信息](#显示状态栏相关的信息)
  + [5.显示应用内存信息](#显示应用内存信息)

------

<span id="基本用法">

## 一、基本用法

<span id="目标设备命令">

### 1.目标设备命令 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb [-d|-e|-s <serialNumber>] <command>
```
> 作用：
```
为命令指定目标设备，如果只有一个设备/模拟器连接，可以直接使用 adb <command>
```
> 示例：
```
先列出设备serialNumber：
$ adb devices
List of devices attached
cf264b8f	device
emulator-5554	device
10.129.164.6:5555	device
再指定设备号进行操作：
adb -s 10.129.164.6:5555 install test.apk
```
> 参数表：

| 参数 | 说明 |
| :--- | :--- |
| -d | 指定当前唯一通过 USB 连接的 Android 设备为命令目标 |
| -e | 指定当前唯一运行的模拟器为命令目标 |
| `-s <serialNumber>` | 指定相应 serialNumber 号的设备/模拟器为命令目标 |

<span id="启动和停止">

### 2.启动和停止 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb start-server
adb kill-server
```
> 作用：
```
启动和停止adb server
```
> 示例：略<br>
> 参数表：略

<span id="查看adb版本">

### 3.查看adb版本 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb version
```
> 作用：
```
查看 adb 版本
```
> 示例：略<br>
> 参数表：略

<span id="获取root权限">

### 4.获取root权限 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb root
```
> 作用：
```
以 root 权限运行 adbd
root用户是系统中唯一的超级管理员，具有系统所有的权限，出于安全原因，安卓将部分命令划分为root权限，只有取得root权限才能执行这些命令。
adb 的运行原理是 PC 端的 adb server 与手机端的守护进程 adbd 建立连接，然后 PC 端的 adb client 通过 adb server 转发命令，adbd 接收命令后解析运行。
所以如果 adbd 以普通权限执行，有些需要 root 权限才能执行的命令无法直接用 adb xxx 执行。这时可以 adb shell 然后 su 后执行命令，也可以让 adbd 以 root 权限执行，这个就能随意执行高权限命令了。
```
> 示例：
```
$ adb root
restarting adbd as root
```
> 参数表：略

<span id="指定网络端口">

### 5.指定网络端口 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb -P <port> start-server
```
> 作用：
```
指定 adb server 的网络端口，默认端口为 5037。
```
> 示例：略<br>
> 参数表：略

------

<span id="设备相关">

## 二、设备相关

<span id="查询已连接设备">

### 1.查询已连接设备 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb devices
```
> 作用：
```
查询已连接设备/模拟器，输出格式为 [serialNumber] [state]
```
> 示例：
```
$ adb devices
List of devices attached
cf264b8f	device
emulator-5554	device
10.129.164.6:5555	device
```
> 参数表：略

<span id="设备型号">

### 2.设备型号 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell getprop ro.product.model
```
> 作用：
```
查看机型
```
> 示例：略<br>
> 参数表：略

<span id="电池状况">

### 3.电池状况 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell dumpsys battery
```
> 作用：
```
查看手机电量状况，其中 scale 代表最大电量，level 代表当前电量。上面的输出表示还剩下 44% 的电量
```
> 示例：略<br>
> 参数表：略

<span id="屏幕分辨率">

### 4.屏幕分辨率 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell wm size
```
> 作用：
```
查看设备屏幕分辨率，如果使用命令修改过，那输出可能是：
Physical size: 1080x1920
Override size: 480x1024
```
> 示例：略<br>
> 参数表：略

<span id="屏幕密度">

### 5.屏幕密度 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell wm density
```
> 作用：
```
查看设备屏幕密度
```
> 示例：略<br>
> 参数表：略

<span id="显示屏参数">

### 6.显示屏参数 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell dumpsys window displays
```
> 作用：
```
显示屏参数，其中 mDisplayId 为 显示屏编号，init 是初始分辨率和屏幕密度，app 的高度比 init 里的要小，
表示屏幕底部有虚拟按键，高度为 1920 - 1794 = 126px 合 42dp。
```
> 示例：
```
$ adb shell dumpsys window displays
WINDOW MANAGER DISPLAY CONTENTS (dumpsys window displays)
  Display: mDisplayId=0
    init=1080x1920 420dpi cur=1080x1920 app=1080x1794 rng=1080x1017-1810x1731
    deferred=false layoutNeeded=false
```
> 参数表：略

<span id="获取设备的android_id">

### 7.获取设备的android_id &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell settings get secure android_id
```
> 作用：
```
获取设备的android_id
```
> 示例：略<br>
> 参数表：略

<span id="IMEI">

### 8.IMEI &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
4.4及以下：adb shell dumpsys iphonesubinfo
5.0及以上：
adb shell
su
service call iphonesubinfo 1
```
> 作用：
```
获取IMEI
```
> 示例：
```
$ adb shell
HWBKL:/ $ su
/system/bin/sh: su: not found
127|HWBKL:/ $ service call iphonesubinfo 1
Result: Parcel(
  0x00000000: 00000000 0000000f 00360038 00390036 '........8.6.6.9.'
  0x00000010: 00340035 00330030 00310037 00370039 '5.4.0.3.7.1.9.7.'
  0x00000020: 00380031 00000034                   '1.8.4...        ')
  把里面的有效内容提取出来就是 IMEI 了，比如这里的是 866954037179184。
```
> 参数表：略

<span id="系统版本">

### 9.系统版本 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell getprop ro.build.version.release
```
> 作用：
```
获取Android 系统版本
```
> 示例：略<br>
> 参数表：略

<span id="获取IP地址">

### 10.获取IP地址 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell ifconfig | grep Mask(Linux)
或
adb shell ifconfig wlan0/rmnet0
```
> 作用：
```
获取 IP 地址
```
> 示例：
```
$ adb shell ifconfig
找到对应rmnet0（手机网络）或wlan0（局域网）下面的inet addr：
inet addr:10.34.106.53  Bcast:10.255.255.255  Mask:255.255.255.255
```
> 参数表：略

<span id="查看网络统计信息">

### 11.查看网络统计信息 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell netstat
```
> 作用：
```
查看网络统计信息
也可以将网络统计信息输出到指定文件：
adb shell netstat><file-path>，如：
adb shell netstat>D:\netstat.log
```
> 示例：
```
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 10.34.106.53:54466      203.209.230.219:https   ESTABLISHED
tcp6       0      0 ::ffff:10.34.106.:51188 :https                  ESTABLISHED
tcp6      24      0 ::ffff:10.34.106.:36910 ::ffff:111.6.252.:https CLOSE_WAIT
tcp6       1      0 ::ffff:10.34.106.:39158 ::ffff:42.186.127.:http CLOSE_WAIT
tcp6      32      0 ::ffff:10.34.106.:60222 ::ffff:59.111.160:https CLOSE_WAIT
tcp6      32      0 ::ffff:10.34.106.:33516 ::ffff:203.119.16:https CLOSE_WAIT
tcp6       0      1 2409:8946:e:dbdb::55924 2404:6800:4008:c04:5228 SYN_SENT
tcp6      32      0 ::ffff:10.34.106.:43874 ::ffff:111.6.252.:https CLOSE_WAIT
tcp6       0      0 ::ffff:10.34.106.:52132 ::ffff:112.13.122:https ESTABLISHED
tcp6       0      0 ::ffff:10.34.106.:60236 ecs-49-4-41-153.co:9877 ESTABLISHED
tcp6       0      0 ::ffff:10.34.106.:43174 tk-in-f188.1e100.n:5228 ESTABLISHED
tcp6       0      0 ::ffff:10.34.106.:41112 ::ffff:111.6.254.:https ESTABLISHED
```
> 参数表：略

<span id="管理网络连接">

### 12.管理网络连接 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell netcfg
```
> 作用：
```
通过配置文件配置和管理网络连接，可以看到网络连接名称、启用状态、IP 地址和 Mac 地址等信息。
adb shell netcfg [<interface> {dhcp|up|down}]
```
> 示例：
```
rmnet_r_ims10 DOWN                                   0.0.0.0/0   0x00001002
rmnet_r_ims00 DOWN                                   0.0.0.0/0   0x00001002
rmnet2   DOWN                                   0.0.0.0/0   0x00001002
ip6tnl0  DOWN                                   0.0.0.0/0   0x00000080
rmnet0   UP                                10.34.106.53/32  0x000010c3
ip_vti0  DOWN                                   0.0.0.0/0   0x00000080
rmnet_tun14 DOWN                                   0.0.0.0/0   0x00001002
p2p0     UP                                     0.0.0.0/0   0x00001003
rmnet_tun04 DOWN                                   0.0.0.0/0   0x00001002
ip6_vti0 DOWN                                   0.0.0.0/0   0x00000080
rmnet_tun12 DOWN                                   0.0.0.0/0   0x00001002
lo       UP                                   127.0.0.1/8   0x00000049
wlan0    UP                                     0.0.0.0/0   0x00001003
rmnet_tun02 DOWN                                   0.0.0.0/0   0x00001002
Hisilicon0 DOWN                                   0.0.0.0/0   0x00001002
rmnet5   DOWN                                   0.0.0.0/0   0x00001002
rmnet_tun10 DOWN                                   0.0.0.0/0   0x00001002
rmnet_r_ims11 DOWN                                   0.0.0.0/0   0x00001002
rmnet_tun00 DOWN                                   0.0.0.0/0   0x00001002
rmnet_r_ims01 DOWN                                   0.0.0.0/0   0x00001002
rmnet3   DOWN                                   0.0.0.0/0   0x00001002
rmnet1   DOWN                                   0.0.0.0/0   0x00001002
rmnet_emc0 DOWN                                   0.0.0.0/0   0x00001002
rndis0   UP                              192.168.42.129/24  0x00001043
rmnet_tun13 DOWN                                   0.0.0.0/0   0x00001002
sit0     DOWN                                   0.0.0.0/0   0x00000080
rmnet_tun03 DOWN                                   0.0.0.0/0   0x00001002
rmnet_ims10 DOWN                                   0.0.0.0/0   0x00001002
rmnet_ims00 UP                                     0.0.0.0/0   0x000010c3
rmnet6   DOWN                                   0.0.0.0/0   0x00001002
rmnet_tun11 DOWN                                   0.0.0.0/0   0x00001002
rmnet_tun01 DOWN                                   0.0.0.0/0   0x00001002
dummy0   UP                                     0.0.0.0/0   0x000000c3
rmnet4   DOWN                                   0.0.0.0/0   0x00001002
```
> 参数表：略

<span id="Mac地址">

### 13.Mac地址 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell cat /sys/class/net/wlan0/address
```
> 作用：
```
这查看的是局域网 Mac 地址，移动网络或其它连接的信息可以通过前面的小节「IP 地址」里提到的 adb shell netcfg 命令来查看。
```
> 示例：略<br>
> 参数表：略

<span id="CPU信息">

### 14.CPU信息 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell cat /proc/cpuinfo
```
> 作用：
```
查看 CPU 信息
```
> 示例：略<br>
> 参数表：略

<span id="内存信息">

### 15.内存信息 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell cat /proc/meminfo
```
> 作用：
```
查看内存信息
```
> 示例：
```
$ adb shell cat /proc/meminfo
MemTotal:        3777284 kB
MemFree:          278204 kB
MemAvailable:     718208 kB
Buffers:            2620 kB
Cached:           700240 kB
SwapCached:        68656 kB
Active:          1197996 kB
Inactive:         699020 kB
Active(anon):     988012 kB
Inactive(anon):   288740 kB
Active(file):     209984 kB
Inactive(file):   410280 kB
Unevictable:       70588 kB
Mlocked:           70588 kB
SwapTotal:       2293756 kB
SwapFree:        1179792 kB
Dirty:                20 kB
Writeback:             0 kB
AnonPages:       1259608 kB
Mapped:           337128 kB
Shmem:             13008 kB
Slab:             389736 kB
SReclaimable:      90316 kB
SUnreclaim:       299420 kB
KernelStack:       63888 kB
PageTables:        76596 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:     4182396 kB
Committed_AS:   84551036 kB
VmallocTotal:   263061440 kB
VmallocUsed:           0 kB
VmallocChunk:          0 kB
CmaTotal:         716800 kB
CmaFree:               0 kB
IonTotalCache:         0 kB
IonTotalUsed:     264340 kB
PActive(anon):         0 kB
PInactive(anon):       0 kB
PActive(file):     73624 kB
PInactive(file):    3140 kB
Isolate1Free:          0 kB
Isolate2Free:          0 kB
RsvTotalUsed:     296452 kB
```
> 参数表：略

<span id="查看Android设备系统属性">

### 16.查看Android设备系统属性 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell getprop <属性名>
```
> 作用：
```
查看 Android 设备系统属性
```
> 示例：
```
$ adb shell getprop ro.product.cpu.abilist
arm64-v8a,armeabi-v7a,armeabi
```
> 参数表：

| 属性名 | 说明 |
| :--- | :--- |
| ro.build.version.sdk | SDK 版本 |
| ro.build.version.release | 	Android 系统版本 |
| ro.build.version.security_patch | Android 安全补丁程序级别 |
| ro.product.model | 	型号 |
| ro.product.brand | 品牌 |
| ro.product.name | 设备名 |
| ro.product.board | 处理器型号 |
| ro.product.cpu.abilist	 | CPU 支持的 abi 列表 |
| persist.sys.isUsbOtgEnabled | 是否支持 OTG |
| dalvik.vm.heapsize | 每个应用程序的内存上限 |
| ro.sf.lcd_density | 屏幕密度 |

------

<span id="修改设置">

## 三、修改设置
注：修改设置之后，运行恢复命令有可能显示仍然不太正常，可以运行 adb reboot 重启设备，或手动重启
修改设置的原理主要是通过 settings 命令修改 /data/data/com.android.providers.settings/databases/settings.db 里存放的设置值。

<span id="分辨率">

### 1.分辨率 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell wm size 480x1024
```
> 作用：
```
表示将分辨率修改为 480px * 1024px。
恢复原分辨率命令：
adb shell wm size reset
```
> 示例：略<br>
> 参数表：略

<span id="屏幕密度">

### 2.屏幕密度 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell wm density 160
```
> 作用：
```
表示将屏幕密度修改为 160dpi。
恢复原屏幕密度命令：
adb shell wm density reset
```
> 示例：略<br>
> 参数表：略

<span id="显示区域">

### 3.显示区域 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell wm overscan 0,0,0,200
```
> 作用：
```
四个数字分别表示距离左、上、右、下边缘的留白像素，以上命令表示将屏幕底部 200px 留白。
恢复原显示区域命令：
adb shell wm overscan reset
```
> 示例：略<br>
> 参数表：略

<span id="关闭USB调试模式">

### 4.关闭USB调试模式 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell settings put global adb_enabled 0
```
> 作用：
```
恢复：
用命令恢复不了了，毕竟关闭了 USB 调试 adb 就连接不上 Android 设备了。
去设备上手动恢复吧：「设置」-「开发者选项」-「Android 调试」。
```
> 示例：略<br>
> 参数表：略

<span id="允许/禁止访问非SDK-API">

### 5.允许/禁止访问非SDK-API &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
允许访问非 SDK API：
adb shell settings put global hidden_api_policy_pre_p_apps 1
adb shell settings put global hidden_api_policy_p_apps 1

禁止访问非 SDK API：
adb shell settings delete global hidden_api_policy_pre_p_apps
adb shell settings delete global hidden_api_policy_p_apps
```
> 作用：
```
允许/禁止访问非 SDK API，不需要设备获得 Root 权限。
```
> 示例：略<br>
> 参数表，命令最后的数字的含义：

| 参数 | 说明 |
| :--- | :--- |
| 0 | 禁止检测非 SDK 接口的调用。该情况下，日志记录功能被禁用，并且令 strict mode API，即 detectNonSdkApiUsage() 无效。不推荐。 |
| 1 | 仅警告——允许访问所有非 SDK 接口，但保留日志中的警告信息，可继续使用 strick mode API。 |
| 2 | 禁止调用深灰名单和黑名单中的接口。 |
| 3 | 禁止调用黑名单中的接口，但允许调用深灰名单中的接口。 |

<span id="状态栏和导航栏的显示隐藏">

### 6.状态栏和导航栏的显示隐藏 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell settings put global policy_control <key-values>
```
> 作用：
```
<key-values> 可由如下几种键及其对应的值组成，格式为 <key1>=<value1>:<key2>=<value2>。
```
> 示例：
```
$ adb shell settings put global policy_control immersive.full=*
表示设置在所有界面下都同时隐藏状态栏和导航栏。
$ adb shell settings put global policy_control immersive.status=com.package1,com.package2:immersive.navigation=apps,-com.package3
表示设置在包名为 com.package1 和 com.package2 的应用里隐藏状态栏，在除了包名为 com.package3 的所有应用里隐藏导航栏。
```
> 参数表：

| key | 说明 |
| :--- | :--- |
| immersive.full | 同时隐藏 |
| immersive.status | 隐藏状态栏 |
| immersive.navigation | 隐藏导航栏 |
| immersive.preconfirms | ? |

| value | 说明 |
| :--- | :--- |
| apps | 所有应用 |
| * | 所有界面 |
| packagename | 指定应用 |
| -packagename | 排除指定应用 |

------

<span id="应用管理">

## 四、应用管理

<span id="查看应用列表">

### 1.查看应用列表 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell pm list packages [-f] [-d] [-e] [-s] [-3] [-i] [-u] [--user USER_ID] [FILTER]
```
> 作用：略
> 示例：略
```
$ adb shell pm list packages
列出所有应用所有应用
```
> 参数表：

| 过滤参数 | 说明 |
| :--- | :--- |
| 无 | 所有应用 |
| -f | 显示应用关联的 apk 文件 |
| -d | 只显示 disabled 的应用 |
| -e | 只显示 enabled 的应用 |
| -s | 只显示系统应用 |
| -3 | 只显示第三方应用 |
| -i | 显示应用的 installer |
| -u | 包含已卸载应用 |
| <FILTER> | 包名包含 FILTER 字符串 |

<span id="按包名关键字查找应用">

### 2.按包名关键字查找应用 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell pm list packages css
```
> 作用：
```
包名包含某字符串的应用，当然也可以使用find来过滤：
adb shell pm list packages | find "css"（Linux用grep css）
```
> 示例：
```
$ adb shell pm list package | find "css"
package:com.css.bcyt
package:com.css.h5shell
package:com.css.videoff
package:com.css.contacts
package:com.huawei.rcsserviceapplication
```
> 参数表：略

<span id="安装APK">

### 3.安装APK &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb install [-lrtsdg] <path_to_apk>，
指定设备安装：
adb -s 设备id install <path_to_apk>
```
> 作用：
```
adb install 后面可以跟一些可选参数来控制安装 APK 的行为，
adb install 实际是分三步完成：
1.push apk 文件到 /data/local/tmp；
2.调用 pm install 安装；
3.删除 /data/local/tmp 下的对应 apk 文件。
```
> 示例：
```
$ adb install app-release.apk
[100%] /data/local/tmp/1.apk
	pkg: /data/local/tmp/1.apk
Success
```
> 参数表：

| 参数 | 说明 |
| :--- | :--- |
| -l | 将应用安装到保护目录 /mnt/asec |
| -r | 允许覆盖安装 |
| -t | 允许安装 AndroidManifest.xml 里 application 指定 android:testOnly="true" 的应用 |
| -s | 将应用安装到 sdcard |
| -d | 允许降级覆盖安装 |
| -g | 授予所有运行时权限 |

<span id="卸载应用">

### 4.卸载应用 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb uninstall [-k] <packagename>
```
> 作用：
```
<packagename> 表示应用的包名，-k 参数可选，表示卸载应用但保留数据和缓存目录。
```
> 示例：
```
$ adb uninstall com.qihoo360.mobilesafe
表示卸载 360 手机卫士。
```
> 参数表：略

<span id="清除应用数据与缓存">

### 5.清除应用数据与缓存 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell pm clear <packagename>
```
> 作用：
```
<packagename> 表示应用名包，这条命令的效果相当于在设置里的应用信息界面点击了「清除缓存」和「清除数据」。
```
> 示例：
```
$ adb shell pm clear com.qihoo360.mobilesafe
表示清除 360 手机卫士的数据和缓存。
```
> 参数表：略

<span id="查看前台Activity">

### 6.查看前台Activity &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
Windows下：
adb shell dumpsys activity activities | findstr mResumedActivity
或
adb shell dumpsys activity activities | find "mResumedActivity"
或
adb shell "dumpsys activity activities | grep mResumedActivity"
Linux下：
adb shell dumpsys activity activities | grep mResumedActivity
```
> 作用：
```
查看前台 Activity
```
> 示例：
```
mResumedActivity: ActivityRecord{3a7ba77 u0 com.huawei.android.launcher/.unihome.UniHomeLauncher t1}
其中的 com.huawei.android.launcher/.unihome.UniHomeLauncher 就是当前处于前台的 Activity。
```
> 参数表：略

<span id="查看正在运行的Services">

### 7.查看正在运行的Services &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell dumpsys activity services [<packagename>]
```
> 作用：
```
<packagename> 参数不是必须的，指定 <packagename> 表示查看与某个包名相关的 Services，不指定表示查看所有 Services。
<packagename> 不一定要给出完整的包名。
```
> 示例：略<br>
> 参数表：略

<span id="查看应用详细信息">

### 8.查看应用详细信息 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell dumpsys package <packagename>
```
> 作用：
```
输出中包含很多信息，包括 Activity Resolver Table、Registered ContentProviders、包名、userId、安装后的文件资源代码等路径、版本信息、权限信息和授予状态、签名版本信息等。
```
> 示例：略<br>
> 参数表：略

<span id="查看应用安装路径">

### 9.查看应用安装路径 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell pm path <PACKAGE>
```
> 作用：
```
输出应用安装路径
```
> 示例：
```
$ adb shell pm path com.tencent.mm
package:/data/app/com.tencent.mm-CGoTnCSqXFHLp66yAN0-zg==/base.apk
```
> 参数表：略

<span id="与应用交互">

### 10.与应用交互 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell am <command>
```
> 作用：
```
主要是使用 am <command> 命令，常用的 <command> 和<Intent>参数在下面，
<INTENT> 参数很灵活，和写 Android 程序时代码里的 Intent 相对应。
```
> 示例：
```
$ adb shell am start -n com.tencent.mm/.ui.LauncherUI
表示调起微信主界面

$ adb shell am start -n org.mazhuang.boottimemeasure/.MainActivity --es "toast" "hello, world"
表示调起 org.mazhuang.boottimemeasure/.MainActivity 并传给它 string 数据键值对 toast - hello, world。

$ adb shell monkey -p com.tencent.mm -c android.intent.category.LAUNCHER 1
不指定Activity名称启动（启动主Activity）
```
> 常用command表：

| command | 说明 |
| :--- | :--- |
| start [options] <INTENT> | 启动 <INTENT> 指定的 Activity |
| startservice [options] <INTENT>	 | 启动 <INTENT> 指定的 Service |
| broadcast [options] <INTENT> | 发送 <INTENT> 指定的广播 |
| force-stop <packagename> | 停止 <packagename> 相关的进程 |

> Intent参数：

| 参数 | 说明 |
| :--- | :--- |
| -a <ACTION> | 指定 action，比如 android.intent.action.VIEW |
| -c <CATEGORY> | 指定 category，比如 android.intent.category.APP_CONTACTS |
| -n <COMPONENT>	 | 指定完整 component 名，用于明确指定启动哪个 Activity，如 com.example.app/.ExampleActivity |

> <INTENT> 里还能带数据，就像写代码时的 Bundle 一样：

| 参数 | 说明 |
| :--- | :--- |
| --esn <EXTRA_KEY> |  |
| --e |  |
| --ez <EXTRA_KEY> <EXTRA_BOOLEAN_VALUE> |  |
| --ei <EXTRA_KEY> <EXTRA_INT_VALUE> |  |
| --el <EXTRA_KEY> <EXTRA_LONG_VALUE> |  |
| --ef <EXTRA_KEY> <EXTRA_FLOAT_VALUE> |  |
| --eu <EXTRA_KEY> <EXTRA_URI_VALUE> |  |
| --ecn <EXTRA_KEY> <EXTRA_COMPONENT_NAME_VALUE> |  |
| --eia <EXTRA_KEY> <EXTRA_INT_VALUE>[,<EXTRA_INT_VALUE...] |  |
| --ela <EXTRA_KEY> <EXTRA_LONG_VALUE>[,<EXTRA_LONG_VALUE...] | long 数组 |

<span id="停止Service">

### 11.停止Service &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell am stopservice [options] <INTENT>
```
> 作用：
```
停止Service
```
> 示例：略<br>
> 参数表：略

<span id="发送广播">

### 12.发送广播 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell am broadcast [options] <INTENT>
```
> 作用：
```
可以向所有组件广播，也可以只向指定组件广播。
这类用法在测试的时候很实用，比如某个广播的场景很难制造，可以考虑通过这种方式来发送广播。
```
> 示例：
```
$ adb shell am broadcast -a android.intent.action.BOOT_COMPLETED
向所有组件广播 BOOT_COMPLETED

$ adb shell am broadcast -a android.intent.action.BOOT_COMPLETED -n org.mazhuang.boottimemeasure/.BootCompletedReceiver
只向 org.mazhuang.boottimemeasure/.BootCompletedReceiver 广播 BOOT_COMPLETED
```
> 预定义广播及正常触发时机：

| action | 触发时机 |
| :--- | :--- |
| android.net.conn.CONNECTIVITY_CHANGE | 网络连接发生变化 |
| android.intent.action.SCREEN_ON | 屏幕点亮 |
| android.intent.action.SCREEN_OFF | 屏幕熄灭 |
| android.intent.action.BATTERY_LOW | 电量低，会弹出电量低提示框 |
| android.intent.action.BATTERY_OKAY | 电量恢复了 |
| android.intent.action.BOOT_COMPLETED | 设备启动完毕 |
| android.intent.action.DEVICE_STORAGE_LOW | 存储空间过低 |
| android.intent.action.DEVICE_STORAGE_OK | 存储空间恢复 |
| android.intent.action.PACKAGE_ADDED | 安装了新的应用 |
| android.net.wifi.STATE_CHANGE | WiFi 连接状态发生变化 |
| android.net.wifi.WIFI_STATE_CHANGED | WiFi 状态变为启用/关闭/正在启动/正在关闭/未知 |
| android.intent.action.BATTERY_CHANGED | 电池电量发生变化 |
| android.intent.action.INPUT_METHOD_CHANGED | 系统输入法发生变化 |
| android.intent.action.ACTION_POWER_CONNECTED | 外部电源连接 |
| android.intent.action.ACTION_POWER_DISCONNECTED | 外部电源断开连接 |
| android.intent.action.DREAMING_STARTED | 系统开始休眠 |
| android.intent.action.DREAMING_STOPPED | 系统停止休眠 |
| android.intent.action.WALLPAPER_CHANGED | 壁纸发生变化 |
| android.intent.action.HEADSET_PLUG | 插入耳机 |
| android.intent.action.MEDIA_UNMOUNTED | 卸载外部介质 |
| android.intent.action.MEDIA_MOUNTED | 挂载外部介质 |
| android.os.action.POWER_SAVE_MODE_CHANGED | 省电模式开启 |

<span id="强制停止应用">

### 13.强制停止应用 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell am force-stop <packagename>
```
> 作用：
```
强制停止应用
```
> 示例：
```
$ adb shell am force-stop com.qihoo360.mobilesafe
表示停止 360 安全卫士的一切进程与服务。
```
> 参数表：略

<span id="收紧内存">

### 14.收紧内存 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell am send-trim-memory  <pid> <level>
```
> 作用：
```
pid: 进程 ID level: HIDDEN、RUNNING_MODERATE、BACKGROUND、 RUNNING_LOW、MODERATE、RUNNING_CRITICAL、COMPLETE
```
> 示例：
```
$ adb shell am send-trim-memory 12345 RUNNING_LOW
表示向 pid=12345 的进程，发出 level=RUNNING_LOW 的收紧内存命令。
```
> 参数表：略

------

<span id="文件管理">

## 五、文件管理

<span id="复制设备里的文件到电脑">

### 1.复制设备里的文件到电脑 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb pull <设备里的文件路径> [电脑上的目录]
```
> 作用：
```
其中 电脑上的目录 参数可以省略，默认复制到当前目录
```
> 示例：
```
$ adb pull /sdcard/sr.mp4 ~/tmp/
```
> 参数表：略

<span id="复制电脑里的文件到设备">

### 2.复制电脑里的文件到设备 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb push <电脑上的文件路径> <设备里的目录>
```
> 作用：
```
复制电脑里的文件到设备
```
> 示例：
```
$ adb push ~/sr.mp4 /sdcard/
```
> 参数表：略

------

<span id="实用功能">

## 六、实用功能

<span id="屏幕截图">

### 1.屏幕截图 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
截图保存到电脑：
adb exec-out screencap -p > sc.png
或
adb shell screencap -p | sed "s/\r$//" > sc.png

截图保存到设备：
adb shell screencap -p /sdcard/sc.png
```
> 作用：
```
可以使用 adb shell screencap -h 查看 screencap 命令的帮助信息，下面是两个有意义的参数及含义：
```
> 示例：略<br>
> 参数表：

| 参数 | 说明 |
| :--- | :--- |
| -p | 指定保存文件为 png 格式 |
| -d display-id	 | 指定截图的显示屏编号（有多显示屏的情况下） |

<span id="录制屏幕">

### 2.录制屏幕 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell screenrecord /sdcard/filename.mp4
```
> 作用：
```
录制屏幕以 mp4 格式保存到 /sdcard，
需要停止时按 Ctrl-C，默认录制时间和最长录制时间都是 180 秒。
```
> 示例：
```
$ adb shell screenrecord /sdcard/filename.mp4
$ adb pull /sdcard/filename.mp4
```
> 参数表：

| 参数 | 说明 |
| :--- | :--- |
| --size WIDTHxHEIGHT | 视频的尺寸，比如 1280x720，默认是屏幕分辨率 |
| --bit-rate RATE | 视频的比特率，默认是 4Mbps |
| --time-limit TIME | 录制时长，单位秒 |
| --verbose | 输出更多信息 |

<span id="重新挂载system分区为可写">

### 3.重新挂载system分区为可写 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
1.进入 shell 并切换到 root 用户权限:
adb shell
su
2.查看当前分区挂载情况：
mount
输出如下：
rootfs / rootfs ro,relatime 0 0
tmpfs /dev tmpfs rw,seclabel,nosuid,relatime,mode=755 0 0
devpts /dev/pts devpts rw,seclabel,relatime,mode=600 0 0
proc /proc proc rw,relatime 0 0
sysfs /sys sysfs rw,seclabel,relatime 0 0
selinuxfs /sys/fs/selinux selinuxfs rw,relatime 0 0
debugfs /sys/kernel/debug debugfs rw,relatime 0 0
none /var tmpfs rw,seclabel,relatime,mode=770,gid=1000 0 0
none /acct cgroup rw,relatime,cpuacct 0 0
none /sys/fs/cgroup tmpfs rw,seclabel,relatime,mode=750,gid=1000 0 0
none /sys/fs/cgroup/memory cgroup rw,relatime,memory 0 0
tmpfs /mnt/asec tmpfs rw,seclabel,relatime,mode=755,gid=1000 0 0
tmpfs /mnt/obb tmpfs rw,seclabel,relatime,mode=755,gid=1000 0 0
none /dev/memcg cgroup rw,relatime,memory 0 0
none /dev/cpuctl cgroup rw,relatime,cpu 0 0
none /sys/fs/cgroup tmpfs rw,seclabel,relatime,mode=750,gid=1000 0 0
none /sys/fs/cgroup/memory cgroup rw,relatime,memory 0 0
none /sys/fs/cgroup/freezer cgroup rw,relatime,freezer 0 0
/dev/block/platform/msm_sdcc.1/by-name/system /system ext4 ro,seclabel,relatime,data=ordered 0 0
/dev/block/platform/msm_sdcc.1/by-name/userdata /data ext4 rw,seclabel,nosuid,nodev,relatime,noauto_da_alloc,data=ordered 0 0
/dev/block/platform/msm_sdcc.1/by-name/cache /cache ext4 rw,seclabel,nosuid,nodev,relatime,data=ordered 0 0
/dev/block/platform/msm_sdcc.1/by-name/persist /persist ext4 rw,seclabel,nosuid,nodev,relatime,data=ordered 0 0
/dev/block/platform/msm_sdcc.1/by-name/modem /firmware vfat ro,context=u:object_r:firmware_file:s0,relatime,uid=1000,gid=1000,fmask=0337,dmask=0227,codepage=cp437,iocharset=iso8859-1,shortname=lower,errors=remount-ro 0 0
/dev/fuse /mnt/shell/emulated fuse rw,nosuid,nodev,relatime,user_id=1023,group_id=1023,default_permissions,allow_other 0 0
/dev/fuse /mnt/shell/emulated/0 fuse rw,nosuid,nodev,relatime,user_id=1023,group_id=1023,default_permissions,allow_other 0 0
找到其中我们关注的带 /system 的那一行：
/dev/block/platform/msm_sdcc.1/by-name/system /system ext4 ro,seclabel,relatime,data=ordered 0 0
3.重新挂载：
mount -o remount,rw -t yaffs2 /dev/block/platform/msm_sdcc.1/by-name/system /system
```
> 作用：
```
/system 分区默认挂载为只读，但有些操作比如给 Android 系统添加命令、删除自带应用等需要对 /system 进行写操作，所以需要重新挂载它为可读写。
需要 root 权限
```
> 示例：略<br>
> 参数表：略

<span id="查看连接过的WiFi密码">

### 4.查看连接过的WiFi密码 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell
su
cat /data/misc/wifi/*.conf
```
> 作用：
```
需要 root 权限。
```
> 示例：
```
network={
	ssid="TP-LINK_9DFC"
	scan_ssid=1
	psk="123456789"
	key_mgmt=WPA-PSK
	group=CCMP TKIP
	auth_alg=OPEN
	sim_num=1
	priority=13893
}

network={
	ssid="TP-LINK_F11E"
	psk="987654321"
	key_mgmt=WPA-PSK
	sim_num=1
	priority=17293
}
ssid 即为我们在 WLAN 设置里看到的名称，psk 为密码，key_mgmt 为安全加密方式。
```
> 参数表：略

<span id="设置系统日期和时间">

### 5.设置系统日期和时间 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell
su
date -s 20160823.131500
```
> 作用：
```
注：需要 root 权限。
```
> 示例：
```
表示将系统日期和时间更改为 2016 年 08 月 23 日 13 点 15 分 00 秒。
```
> 参数表：略

<span id="重启手机">

### 6.重启手机 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb reboot
```
> 作用：略<br>
> 示例：略<br>
> 参数表：略

<span id="检测设备是否已root">

### 7.检测设备是否已root &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell
su
```
> 作用：
```
此时命令行提示符是 $ 则表示没有 root 权限，是 # 则表示已 root。
```
> 示例：略<br>
> 参数表：略

<span id="使用Monkey进行压力测试">

### 8.使用Monkey进行压力测试 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell monkey -p <packagename> -v 500
表示向 <packagename> 指定的应用程序发送 500 个伪随机事件
```
> 作用：
```
Monkey 可以生成伪随机用户事件来模拟单击、触摸、手势等操作，可以对正在开发中的程序进行随机压力测试。
```
> 示例：略<br>
> 参数表：略

<span id="开启/关闭WiFi">

### 9.开启/关闭WiFi &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
开启 WiFi：
adb root
adb shell svc wifi enable

关闭 WiFi：
adb root
adb shell svc wifi disable
```
> 作用：
```
若执行成功，输出为空；若未取得 root 权限执行此命令，将执行失败，输出 Killed。
```
> 示例：略<br>
> 参数表：略

------

<span id="模拟按键和输入">

## 七、模拟按键和输入

<span id="电源键">

### 1.电源键 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell input keyevent 26
```
> 作用：
```
相当于按电源键。
```
> 示例：略<br>
> 参数表：略

<span id="菜单键">

### 2.菜单键 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell input keyevent 82
```
> 作用：略<br>
> 示例：略<br>
> 参数表：略

<span id="HOME键">

### 3.HOME键 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell input keyevent 3
```
> 作用：略<br>
> 示例：略<br>
> 参数表：略

<span id="返回键">

### 4.返回键 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell input keyevent 4
```
> 作用：略<br>
> 示例：略<br>
> 参数表：略

<span id="音量控制">

### 5.音量控制 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
增加音量：
adb shell input keyevent 24

降低音量：
adb shell input keyevent 25

静音：
adb shell input keyevent 164
```
> 作用：略<br>
> 示例：略<br>
> 参数表：略

<span id="媒体控制">

### 6.媒体控制 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
播放/暂停：
adb shell input keyevent 85

停止播放：
adb shell input keyevent 86

播放下一首：
adb shell input keyevent 87

播放上一首：
adb shell input keyevent 88

恢复播放：
adb shell input keyevent 126

暂停播放：
adb shell input keyevent 127
```
> 作用：略<br>
> 示例：略<br>
> 参数表：略

<span id="点亮/熄灭屏幕">

### 7.点亮/熄灭屏幕 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
点亮屏幕：
adb shell input keyevent 224

熄灭屏幕：
adb shell input keyevent 223
```
> 作用：略<br>
> 示例：略<br>
> 参数表：略

<span id="滑动解锁">

### 8.滑动解锁 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell input swipe 300 1000 300 500
参数 300 1000 300 500 分别表示起始点x坐标 起始点y坐标 结束点x坐标 结束点y坐标。
```
> 作用：
```
如果锁屏没有密码，是通过滑动手势解锁，那么可以通过 input swipe 来解锁
```
> 示例：略<br>
> 参数表：略

<span id="输入文本">

### 9.输入文本 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell input text hello
```
> 作用：
```
在焦点处于某文本框时，可以通过 input 命令来输入文本。
```
> 示例：略<br>
> 参数表：

| keycode | 说明 |
| :--- | :--- |
| 3 | HOME 键 |
| 4 | 返回键 |
| 5 | 打开拨号应用 |
| 6 | 挂断电话 |
| 24 | 增加音量 |
| 25 | 降低音量 |
| 26 | 电源键 |
| 27 | 拍照（需要在相机应用里） |
| 64 | 打开浏览器 |
| 82 | 菜单键 |
| 85 | 播放/暂停 |
| 86 | 停止播放 |
| 87 | 播放下一首 |
| 88 | 播放上一首 |
| 122 | 移动光标到行首或列表顶部 |
| 123 | 移动光标到行末或列表底部 |
| 126 | 恢复播放 |
| 127 | 暂停播放 |
| 164 | 静音 |
| 176 | 打开系统设置 |
| 187 | 切换应用 |
| 207 | 打开联系人 |
| 208 | 打开日历 |
| 209 | 打开音乐 |
| 210 | 打开计算器 |
| 220 | 降低屏幕亮度 |
| 221 | 提高屏幕亮度 |
| 223 | 系统休眠 |
| 224 | 点亮屏幕 |
| 231 | 打开语音助手 |
| 276 | 如果没有 wakelock 则让系统休眠 |

------

<span id="查看日志">

## 八、查看日志
Android 系统的日志分为两部分，底层的 Linux 内核日志输出到 /proc/kmsg，Android 的日志输出到 /dev/log。

<span id="Android日志">

### 1.Android日志 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
[adb] logcat [options] [filterspecs]
```
> 作用：
```
其中filterspecs是一系列的<tag>[:priority]的组合，tag是指标签关键字（如果是*则指全部,），如果没有tag，则输出全部，而priority则是指过滤的优先级
```
> 示例：
```
见下
```
> 参数表：

| 常用options | 说明 | 用例 |
| :--- | :--- | :--- |
| -b <buffer> | 加载一个可使用的日志缓冲区供查看，比如event和radio。默认值是main。 注：缓冲区类型（radio-无线缓冲区，events-事件缓冲区，main-主缓冲区，默认） | - |
| -c | 清除缓冲区中的全部日志并退出（清除完后可以使用-g查看缓冲区） | adb logcat -c |
| -d | 将缓冲区的log转存到屏幕中然后退出 | adb logcat -d |
| -f <filename> | 将log输出到指定的文件中<文件名>.默认为标准输出（stdout） | adb logcat -f /data/local/tmp/log.txt -n 10 -r 1 |
| -g | 打印日志缓冲区的大小并退出 | adb logcat -g |
| -n <count> | 设置日志的最大数目<count>，默认值是4，需要和-r选项一起使用 | - |
| -r <kbytes> | 没<kbytes>时输出日志，默认值是16，需要和-f选项一起使用 | - |
| -s | 设置过滤器 | - |
| -v <format> | 设置输出格式的日志消息。默认是短暂的格式。支持的格式列表 | - |

| 日志纪录方法priority | 说明 |
| :--- | :--- |
| V | Verbose (default for <tag>)，优先级最低，输出得最多 |
| D | (default for '*') |
| I | Info |
| W | Warn |
| E | Error |
| F | Fatal |
| S | Silent (suppress all output)，优先级最高，啥也不输出 |

<span id="示例1：按优先级过滤">

### 2.示例1：按优先级过滤 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb logcat *:W
```
> 作用：
```
按W优先级过滤日志，则会将该级别及以上的日志输出
会将 Warning、Error、Fatal 和 Silent 日志输出。
```
> 示例：略<br>
> 参数表：略

<span id="示例2：按tag和级别过滤日志">

### 3.示例2：按tag和级别过滤日志 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb logcat ActivityManager:I MyApp:D *:S
```
> 作用：
```
表示输出 tag ActivityManager 的 Info 以上级别日志，输出 tag MyApp 的 Debug 以上级别日志，及其它 tag 的 Silent 级别日志（即屏蔽其它 tag 日志）。
```
> 示例：略<br>
> 参数表：略

<span id="示例3：显示某一TAG的日志信息">

### 4.示例3：显示某一TAG的日志信息 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb logcat -s TAG名称
```
> 作用：
```
显示某一TAG的日志信息
```
> 示例：略<br>
> 参数表：略

<span id="示例4：清理已经存在的日志">

### 5.示例4：清理已经存在的日志 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb logcat -c
```
> 作用：
```
清理已经存在的日志
```
> 示例：略<br>
> 参数表：略

<span id="示例5：将日志显示在控制台后退出">

### 6.示例5：将日志显示在控制台后退出 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb logcat -d
```
> 作用：
```
将日志显示在控制台后退出
```
> 示例：略<br>
> 参数表：略

<span id="logcat格式化输出">

### 7.logcat格式化输出 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb logcat -v <format>
```
> 作用：
```
日志消息包含一个元数据字段，除了标签和优先级，还可以修改输出显示一个特定的元数据字段格式的消息。为此，需要使用-v选项来指定一个支持的输出格式。下表为支持的格式
```
> 示例：
```
adb logcat -v long ActivityManager:I *:S
```
> 参数表：

| 格式名 | 说明 | 格式 | 示例 |
| :--- | :--- | :--- | :--- |
| brief | 显示优先级/标记和过程的PID发出的消息（默认格式） | <priority>/<tag>(<pid>): <message> | D/HeadsetStateMachine( 1785): Disconnected process message: 10, size: 0 |
| process | 只显示PID | <priority>(<pid>) <message> | D( 1785) Disconnected process message: 10, size: 0  (HeadsetStateMachine) |
| tag | 只显示优先级/标记 | <priority>/<tag>: <message> | D/HeadsetStateMachine: Disconnected process message: 10, size: 0 |
| raw | 显示原始的日志消息，没有其他元数据字段 | <message> | Disconnected process message: 10, size: 0 |
| time | 调用显示日期、时间、优先级/标签和过程的PID发出消息 | <datetime> <priority>/<tag>(<pid>): <message> | 08-28 22:39:39.974 D/HeadsetStateMachine( 1785): Disconnected process message: 10, size: 0 |
| threadtime | 调用显示日期、时间、优先级、标签遗迹PID TID线程发出的消息 | <datetime> <pid> <tid> <priority> <tag>: <message> | 08-28 22:39:39.974  1785  1832 D HeadsetStateMachine: Disconnected process message: 10, size: 0 |
| long | 显示所有元数据字段与空白行和单独的消息 | [ <datetime> <pid>:<tid> <priority>/<tag> ]
<message> | [ 08-28 22:39:39.974  1785: 1832 D/HeadsetStateMachine ]
Disconnected process message: 10, size: 0 |

<span id="内核日志">

### 8.内核日志 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell dmesg
```
> 作用：
```
通过内核日志我们可以做一些事情，比如衡量内核启动时间，在系统启动完毕后的内核日志里找到 Freeing init memory 那一行前面的时间就是。
```
> 示例：
```
$ adb shell dmesg
<6>[14201.684016] PM: noirq resume of devices complete after 0.982 msecs
<6>[14201.685525] PM: early resume of devices complete after 0.838 msecs
<6>[14201.753642] PM: resume of devices complete after 68.106 msecs
<4>[14201.755954] Restarting tasks ... done.
<6>[14201.771229] PM: suspend exit 2016-08-28 13:31:32.679217193 UTC
<6>[14201.872373] PM: suspend entry 2016-08-28 13:31:32.780363596 UTC
<6>[14201.872498] PM: Syncing filesystems ... done.
```
> 参数表：略

------

<span id="刷机相关命令">

## 九、刷机相关命令

<span id="重启到Recovery模式">

### 1.重启到Recovery模式 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb reboot recovery
```
> 作用：略<br>
> 示例：略<br>
> 参数表：略

<span id="从Recovery重启到Android">

### 2.从Recovery重启到Android &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb reboot
```
> 作用：略<br>
> 示例：略<br>
> 参数表：略

<span id="重启到Fastboot模式">

### 3.重启到Fastboot模式 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb reboot bootloader
```
> 作用：略<br>
> 示例：略<br>
> 参数表：略

<span id="通过sideload更新系统">

### 4.通过sideload更新系统 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
1.重启到 Recovery 模式：
adb reboot recovery

2.在设备的 Recovery 界面上操作进入 Apply update-Apply from ADB
注：不同的 Recovery 菜单可能与此有差异，有的是一级菜单就有 Apply update from ADB。

3.通过 adb 上传和更新系统：
adb sideload <path-to-update.zip>
```
> 作用：
```
如果我们下载了 Android 设备对应的系统更新包到电脑上，那么也可以通过 adb 来完成更新。
```
> 示例：略<br>
> 参数表：略

------

<span id="安全相关命令">

## 十、安全相关命令

<span id="启用/禁用SELinux">

### 1.启用/禁用SELinux &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
启用 SELinux：
adb root
adb shell setenforce 1

禁用 SELinux：
adb root
adb shell setenforce 0
```
> 作用：略<br>
> 示例：略<br>
> 参数表：略

<span id="启用/禁用dm_verity">

### 2.启用/禁用dm_verity &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
启用 dm_verity：
adb root
adb enable-verity

禁用 dm_verity：
adb root
adb disable-verity
```
> 作用：略<br>
> 示例：略<br>
> 参数表：略

------

<span id="adb-shell">

## 十一、adb shell
Android 系统是基于 Linux 内核的，所以 Linux 里的很多命令在 Android 里也有相同或类似的实现，在 adb shell 里可以调用

<span id="查看进程">

### 1.查看进程 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell ps
```
> 作用：
```
查看进程
```
> 示例：
```
USER     PID   PPID  VSIZE  RSS     WCHAN    PC        NAME
root      1     0     8904   788   ffffffff 00000000 S /init
root      2     0     0      0     ffffffff 00000000 S kthreadd
...
u0_a71    7779  5926  1538748 48896 ffffffff 00000000 S com.sohu.inputmethod.sogou:classic
u0_a58    7963  5926  1561916 59568 ffffffff 00000000 S org.mazhuang.boottimemeasure
...
shell     8750  217   10640  740   00000000 b6f28340 R ps
```
> 各列含义：

| 列名 | 说明 |
| :--- | :--- |
| USER | 所属用户 |
| PID | 进程 ID |
| PPID | 父进程 ID |
| NAME | 进程名 |

<span id="查看实时资源占用情况">

### 2.查看实时资源占用情况 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell top
```
> 作用：
```
查看实时资源占用情况
```
> 示例：
```
User 0%, System 6%, IOW 0%, IRQ 0%
User 3 + Nice 0 + Sys 21 + Idle 280 + IOW 0 + IRQ 0 + SIRQ 3 = 307

  PID PR CPU% S  #THR     VSS     RSS PCY UID      Name
 8763  0   3% R     1  10640K   1064K  fg shell    top
  131  0   3% S     1      0K      0K  fg root     dhd_dpc
 6144  0   0% S   115 1682004K 115916K  fg system   system_server
  132  0   0% S     1      0K      0K  fg root     dhd_rxf
 1731  0   0% S     6  20288K    788K  fg root     /system/bin/mpdecision
  217  0   0% S     6  18008K    356K  fg shell    /sbin/adbd
 ...
 7779  2   0% S    19 1538748K  48896K  bg u0_a71   com.sohu.inputmethod.sogou:classic
 7963  0   0% S    18 1561916K  59568K  fg u0_a58   org.mazhuang.boottimemeasure
 ...
```
> 各列含义：

| 列名 | 说明 |
| :--- | :--- |
| PID | 进程 ID |
| PR | 优先级 |
| CPU% | 当前瞬间占用 CPU 百分比 |
| S | 进程状态（R=运行，S=睡眠，T=跟踪/停止，Z=僵尸进程） |
| #THR | 线程数 |
| VSS | Virtual Set Size 虚拟耗用内存（包含共享库占用的内存） |
| RSS | Resident Set Size 实际使用物理内存（包含共享库占用的内存） |
| PCY | 调度策略优先级，SP_BACKGROUND/SPFOREGROUND |
| UID | 进程所有者的用户 ID |
| NAME | 进程名 |

<span id="显示activity相关的信息">

### 3.显示activity相关的信息 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell dumpsys activity
```
> 作用：
```
显示activity相关的信息
```
> 示例：略<br>
> 参数表：略

<span id="显示状态栏相关的信息">

### 4.显示状态栏相关的信息 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell dumpsys statusbar
```
> 作用：
```
显示状态栏相关的信息
```
> 示例：略<br>
> 参数表：略

<span id="显示应用内存信息">

### 5.显示应用内存信息 &#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;&#8195;[回到目录](#目录)
> 命令：
```
adb shell dumpsys meminfo $package_name or $pid
```
> 作用：
```
使用程序的包名或者进程id显示内存信息
```
> 示例：略<br>
> 参数表：略
