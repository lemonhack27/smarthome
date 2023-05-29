# smarthome
## 环境配置
###  Qt安装
* 下载qt5.12.9并安装（高版本不用单独下载qt creator和qt，已经内置了）
```
wget https://download.qt.io/archive/qt/5.12/5.12.9/qt-opensource-linux-x64-5.12.9.run
chmod +x qt-opensource-linux-x64-5.12.9.run
sudo ./qt-opensource-linux-x64-5.12.9.run
```
* 在选择qt组件的时候，除了安卓开头的选项不选，其他都选中
* 由于 Qt5.0 的库默认会链接到 OpenGL，但是在 Ubuntu 机器上没有安装OpenGL，所以们需要在 Ubuntu 下安装 OpenGL Library。在 Ubuntu 终端下输入如下指令。
```
sudo apt-get install libglu1-mesa-dev
```
* 同时安装了qt后需要设置qmake的环境变量。
```
sudo vi /etc/profile
```
* 在etc/profile中声明环境变量，在文件末尾键入以下声明
```
export QTDIR=/opt/Qt5.12.9/5.12.9
export PATH=$QTDIR/gcc_64/bin:$PATH
export LD_LIBRARY_PATH=$QTDIR/gcc_64/lib:$LD_LIBRARY_PATH
```

### Qt交叉编译器安装
* 交叉编译器使用的是正点原子提供的，文件位置为**A-基础资料\5、开发工具\1、交叉编译器\st-example-image-qtwayland-openstlinux-weston-stm32mp1-x86_64-toolchain-3.1-snapshot.sh**
* 拷贝文件到ubuntu下，赋予此脚本可执行权限，同时执行脚本。执行脚本过程中直接键入enter选择默认选项
```
chmod +x st-example-image-qtwayland-openstlinux-weston-stm32mp1-x86_64-toolchain-3.1-snapshot.sh
./st-example-image-qtwayland-openstlinux-weston-stm32mp1-x86_64-toolchain-3.1-snapshot.sh
```
* __关于交叉编译工具的使用：__
* 1、使能交叉编译器：在使用交叉编译器前一定要使能环境变量，在不同终端或者切换用户时需要重新使能环境变量才能使用。
```
source /opt/st/stm32mp1/3.1-snapshot/environment-setup-cortexa7t2hf-neon-vfpv4-ostl-linux-gnueabi
```
* 2、查看生效的环境变量。执行 env 就可以看到当前生效的环境变量。内容很多，这里我们主要看下环境变量 CC 的值。
```
env
```
* 3、查看 gcc 版本。执行以下指令查看 gcc 版本，这里我们的 gcc 版本是 9.3.0 版本。
```
arm-ostl-linux-gnueabi-gcc --version
```
### 配置Qt Creator Kits
* Kits 译作套件，Qt Creator Kits 也就是开发编译环境套件，我们可以搭建不同平台的套件，用不同的套件编译出不同平台的应用程序，这也就是 Qt 跨平台的特性。我们需要在 Qt 启动脚本里写入设置使能环境变量的指令，首先打开此脚本。
```
sudo vi /opt/Qt5.12.9/Tools/QtCreator/bin/qtcreator.sh
```
在 qtcreator.sh 里的第一行插入如下指令然后保存
```
source /opt/st/stm32mp1/3.1-snapshot/environment-setup-cortexa7t2hf-neon-vfpv4-ostl-linux-gnueabi
```
* 打开Qt Creator，在主界面打开上方工具栏：打开 Tool（工具），打开 Options（选项）。
* 配置 Qt Versions，点击添加，选择交叉编译工具链路径下的 qmake，__路径为/opt/st/stm32mp1/3.1-snapshot/sysroots/x86_64-ostl_sdk-linux/usr/bin/qmake__ ， 用 于 生 成Makefile，以此编译程序。同时改版本名称为 __ATK-MP157__。
* 打开“编译器”选项卡，添加 C++。配置编译器，编译器的路径为交叉编译器。__路径为/opt/st/stm32mp1/3.1-snapshot/sysroots/x86_64-ostl_sdk-linux/usr/bin/arm-ostl-linux-gnueabi/arm-ostl-linux-gnueabi-g++__。将“名称”改为 __ATK-MP157-GCC__。
* 配置 Kits，点击**添加**，将“**名称**”改为 ATK-MP157。**Compiler**中C：< No compiler>，C++：ATK-MP157-GCC。**Qt version**为ATK-MP157，在 **Qt mkspec** 处手动写上“linux-oe-g++”。不要漏了其中一项，否则可能编译出错。

### 远程调试Qt程序
* 在 QtCreator 里默认情况下，会使用 sftp 或 rsync 发送程序到开发板。正点原子最新的出厂系统里配置了 sftp 和 rsync，可以使用它们来远程调试 Qt 程序（前提是开发板和虚拟机能 ping通。
* 打开菜单栏的“工具”，打开“选项”。点击设备，添加通用 Linux 设备。填写设备备名为 MP157，设备 IP 地址填写自己的开发板 IP 地址，最后填写用户名 root。点击下一步。验证设备连接性，这里点击 close 关闭即可。
* 返回到设备选项卡中，将验证类型改为 Default。在 Kits 选项卡中将套件的设备改为刚刚创建的 MP157。确保当前是选择 ATK-MP157 套件 Debug 模式构建。选中项目，右键直接构建，然后运行即可，在开发板上就会出厂对应的 Qt 界面。
* Eglfs 方式验证 Qt 程序
```
systemctl stop atk-qtapp-start.service //暂时关闭出厂系统桌面
chmod 777 test //赋予测试文件权限
./test -platform eglfs //以 eglfs 方式运行测试程序
```
* Linuxfb 方式验证 Qt 程序(默认)
```
systemctl stop atk-qtapp-start.service // 暂时停止 Qt 桌面服务，重启服务用 restart
chmod 777 test //赋予测试文件权限
./test -platform linuxfb //以 linuxfb 方式运行测试程序
```
## MQTT移植
### MQTT服务器安装
* 选择EMQ X作为MQTT的服务器，因为这是个开源平台，并且具有可视化的后台管理，便于开发。EMQ X[下载地址官网](https://www.emqx.com/zh/try?product=enterprise)，EMQ X[使用指南](https://docs.emqx.com/zh/enterprise/v4.4/)
* 1、下载
```
wget https://www.emqx.com/zh/downloads/enterprise/4.4.18/emqx-ee-4.4.18-otp24.3.4.2-1-ubuntu18.04-amd64.zip
```
* 2、安装
```
unzip emqx-ee-4.4.18-otp24.3.4.2-1-ubuntu18.04-amd64.zip
```
* 3、运行
```
./emqx/bin/emqx start
```
* 4、连接查看
web端输入http://192.168.6.130:18083，__192.168.6.130为自己的IP，端口默认为18083__
用户名开始都是admin 密码默认为：pubilc。
### MQTT.fx客户端连通性测试
* 1、Broker Address：服务器IP地址
* 2、Broker Port：1883（本地端口）
* 3、connect
### MQTT源码下载
* 下载的MQTT源码的版本需要和QT版本对应，否则**编译不通过**。[下载地址](https://gitcode.net/mirrors/qt/qtmqtt/-/tree/5.12.9)。
### MQTT源码移植
* 1、下载后的MQTT源码文件夹名称为qtmqtt-5.12.9。在环境配置章，QT安装地址为/opt/Qt5.12.9。以下将在这两个个文件夹操作。
* 2、在/opt/Qt5.12.9/5.12.9/gcc_64/include文件下创建QtMqtt文件夹
```
sudo mkdir QtMqtt
```
* 3、将复制qtmqtt-5.12.9中的头文件复制到QtMqtt文件夹中
```
sudo qtmqtt-5.12.9/src/mqtt/*.h /opt/Qt5.12.9/5.12.9/gcc_64/include/QtMqtt
```
* 4、使用QT打开MQTT源码文件夹中的工程，然后点击构建，对源码进行编译，会在qtmqtt-5.12.9同级目录输出编译文件**build-qtmqtt-Desktop_Qt_5_12_9_GCC_64bit-Debug**
* 5、复制编译文件夹中生成的头文件依赖到qtmqtt-5.12.9/src/mqtt文件夹中
```
cp build-qtmqtt-Desktop_Qt_5_12_9_GCC_64bit-Debug/include qtmqtt-5.12.9/src/mqtt -r
```
* 6、新建文件夹，用于自己的MQTT工程存放
```
mkdir MyMqttServer
cd MyMqttServer
```
* 7、在QT中新建工程，位置为第六步新建的文件夹，工程名称设为MqttServer。
* 8、将编译生成的库文件夹和qtmqtt-5.12.9/src/mqtt文件夹复制到自己MQTT的QT工程的同级目录
```
cp qtmqtt-5.12.9/src/mqtt build-qtmqtt-Desktop_Qt_5_12_9_GCC_64bit-Debug/lib MyMqttServer/MqttServer -r
```
* 9、将第八步复制的库文件作为外部库添加到自己的MQTT工程里面。步骤：在QT中右击工程->添加库->选择外部库->添加库文件（MyMqttServer/MqttServer/lib/libQt5Mqtt.so），包含路径（MyMqttServer/MqttServer/lib）。同时取消勾选Mac、Windows，只勾选Linux。
* 10、在头文件目录(Headers)添加第八步复制的头文件。步骤：右击Header->路径位置为MyMqttServer/MqttServer/mqtt/include/QtMqtt/QtMqttDepends，双击QtMqttDepends文件添加即可。至此，MQTT在Qt平台移植完成。

