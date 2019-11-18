# ADB-About
Android Debug Bridge Online document

---------

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

## ADB架构
ADB是一个C/S架构的应用程序，由三部分组成：
1. 运行在pc端的adb client：<br>
命令行程序”adb”用于从shell或脚本中运行adb命令。首先，“adb”程序尝试定位主机上的ADB服务器，如果找不到ADB服务器，“adb”程序自动启动一个ADB服务器。接下来，当设备的adbd和pc端的adb server建立连接后，adb client就可以向ADB servcer发送服务请求；
2. 运行在pc端的adb server：<br>
ADB Server是运行在主机上的一个后台进程。它的作用在于检测USB端口感知设备的连接和拔除，以及模拟器实例的启动或停止，ADB Server还需要将adb client的请求通过usb或者tcp的方式发送到对应的adbd上；
3. 运行在设备端的常驻进程adb demon (adbd)：<br>
程序“adbd”作为一个后台进程在Android设备或模拟器系统中运行。它的作用是连接ADB服务器，并且为运行在主机上的客户端提供一些服务；

## 一、基本用法
### 1. 目标设备命令
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
> 参数表<br>

| 参数 | 说明 |
| :--- | :--- |
| -d | 指定当前唯一通过 USB 连接的 Android 设备为命令目标 |
| -e | 指定当前唯一运行的模拟器为命令目标 |
| `-s <serialNumber>` | 指定相应 serialNumber 号的设备/模拟器为命令目标 |
