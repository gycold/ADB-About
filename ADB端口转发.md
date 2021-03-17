### 一、adb 端口映射

#### 原理：

1. **forward**

正向代理，将本地PC指定的Port端口，映射到设备手机指定的Port端口上，以便解决 PC -> Phone 的访问问题。

顾名思义，`adb forward`的功能是建立一个转发，例如：`adb forward tcp:11111 tcp:22222`，是将PC端的`11111`端口收到的数据，转发给到手机中`22222`端口。但是光执行这个命令还不能转发数据，还需要完成两个步骤才能传数据。这两个步骤是：

（a）在手机端，建立一个端口为`22222`的server，并打开server到监听状态。
（b）在PC端，建立一个`socket client`端，连接到端口为`11111`的server上。这两个步骤有先后顺序，步骤（a）要先执行。
`adb forward tcp:11111 tcp:22222` 可以在步骤(a)之前执行。

之后，PC端访问`127.0.0.1:11111`即可与Phone通信

![img](https://img-blog.csdn.net/2018042209420611?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM1NTM1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2. **reverse**

反向代理，将Phone指定端口，映射到PC指定的端口上，解决Phone -> PC的访问问题。用于以下场景：

- Phone 无法访问某个 第三方服务Server服务器
- PC 却可以访问该 第三方服务Server服务器
- 然后可以Phone -> PC 通过PC来访问 第三方服务Server服务器

#### forward基本操作

1. `adb forward <spec-local> <spec-remote>`，socket定向绑定，一般spec-local指PC端口，spec-remote指手机端口

   spec格式为：

   + tcp:<port>
   + localabstract:<unix domain socket name>
   + localreserved:<unix domain socket name>
   + localfilesystem:<unix domain socket name>
   + dev:<character device name>
   + jdwp:<process pid> (remote only)

   示例：`adb forward tcp:11111 tcp:22222`，执行结果：111111

   把PC电脑端TCP端口11111 的数据转发到与电脑通过adb连接的Android设备的TCP端口22222上，换言之，假设现在PC端在端口11111 绑定，如果在该端口读写数据的话，那么数据将会被转发到Android设备指定的端口22222上。此时pad担任server端角色，pc担任client角色。

2. `adb forward --list`

   显示所有从设备发出的反向socket连接

   执行结果：`1982bc33 tcp:11111 tcp:22222`

3. `adb forward --remove <spec-local>`，移除指定的定向绑定

   示例：`adb forward --remove  tcp:11111`

4. `adb forward --remove-all`，移除所有定向绑定

5. `adb forward —no-rebind <spec-local> <spec-remote>`，映射端口连接，同1，但是如果local已经映射了就会失败

6. `adb forward --no-rebind tcp:6100 tcp:7100`，映射端口连接，同1，但是如果local已经映射了就会失败

#### reverse基本操作

1. `adb reverse <spec-remote> <spec-local>`，反向socket绑定，一般spec-remote指手机端口，spec-local指PC端口

   spec格式为：

   + tcp:<port>
   + localabstract:<unix domain socket name>
   + localreserved:<unix domain socket name>
   + localfilesystem:<unix domain socket name>

   示例：`adb forward tcp:12580 tcp:18888`，执行结果：12580

2. `adb reverse --list`，显示所有从设备发出的反向socket连接

3. `adb reverse --remove <spec-remote>`，移除反向绑定

4. `adb reverse --remove-all`，移除所有反向绑定

5. `adb reverse —no-rebind (remote) (local)`，映射端口连接，同1，但是如果local已经映射了就会失败

6. `adb reverse --no-rebind tcp:7000 tcp:5000`，映射端口连接，同1，但是如果local已经映射了就会失败





