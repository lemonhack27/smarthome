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
