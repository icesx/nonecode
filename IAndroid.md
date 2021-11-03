android
============
## 安装

### 关于翻墙

https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/

### prepare
+ repo
``` shell
	mkdir ~/bin
	PATH=~/bin:$PATH
	curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
	chmod a+x ~/bin/repo
```
+ vi repo 
REPO_URL = 'https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
+ install lib
```
sudo apt-get install m4 ccache
```

+ download code
```
wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar # 下载初始化包——如果无法翻墙的话
tar xf aosp-latest.tar
cd AOSP   # 解压得到的 AOSP 工程目录

# 这时 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录
repo sync # 正常同步一遍即可得到完整目录
# 或 repo sync -l 仅checkout代码
```
+ 调整编译缓存
	ccache -M 50G
	```
	CCache 是一个能够把编译的中间产物缓存起来的工具，它会在实际编译之前先检查缓存
	可以通过如下命令查看信息。
	ccache -s
	```
+ about gcc
	android的交叉编译 工具链在:/prebuilt/gcc/linux-x86/arm-eabi-4.2.1/bin
+ android 编译用的 toochain可以在envsetup.sh中查看到
	toolchaindir=arm/arm-linux-androideabi-$targetgccversion/bin
## build
```
source ./build/envsetup.sh 
lunch  aosp_arm64-eng 
make -j8
```
## about make 
```
Make 目标          说明
make clean           执行清理，等同于：rm -rf out/。
make sdk             编译出 Android 的 SDK。
make clean-sdk       清理 SDK 的编译产物。
make update-api      更新 API。在 framework API 改动之后，需要首先执行该命令来更新 API，公开的 API 记录在 frameworks/base/api 目录下。
make dist            执行 Build，并将 MAKECMDGOALS 变量定义的输出文件拷贝到 /out/dist 目录。
make all              编译所有内容，不管当前产品的定义中是否会包含。
make help             帮助信息，显示主要的 make 目标。
make snod             从已经编译出的包快速重建系统镜像。（其实主要对system进行打包）
make libandroid_runtime   编译所有 JNI framework 内容。
make framework        编译所有 Java framework 内容。
make services         编译系统服务和相关内容。
make <local_target>   编译一个指定的模块，local_target 为模块的名称。
make clean-<local_target>   清理一个指定模块的编译结果。
make dump-products     显示所有产品的编译配置信息，例如：产品名，产品支持的地区语言，产品中会包含的模块等信息。
make bootimage        生成 boot.img
make recoveryimage   生成 recovery.img
make userdataimage   生成 userdata.img
make cacheimage       生成 cache.img

```

## tinkerboard
### 下载源代码android-6.0.1
```
repo init --config-name
repo init -u https://git@bitbucket.org/TinkerBoard_Android/manifest.git -b sbc/tinkerboard/asus/Android-6.0.1
repo sync

```
### 下载源代码android-7.0.1
```
repo init --config-name
repo init -u https://git@bitbucket.org/TinkerBoard_Android/manifest.git -b sbc/tinkerboard/asus/Android-7.1.2
repo sync

```
### 编译u-boot

```
apt install bc
cd [source tree]/u-boot
make rk3288_defconfig
make
```
### 编译kernel
```
sudo apt install lzop gcc-multilib
A.cd [source tree]/kernel
B.make rockchip_defconfig
C.make rk3288-miniarm.img
```
### 编译android
```
sudo add-apt-repository ppa:openjdk-r/ppa  
sudo apt-get update   
sudo apt-get install openjdk-7-jdk  

sudo apt install libxml2-utils
A.cd [source tree]
B.source build/envsetup.sh
C.lunch rk3288-userdebug
D.make -j8
E../mkimage.sh
```
### 统一固件
```
A.cd [source tree]/RKTools/linux/Linux_Pack_Firmware/rockdev_rk3288
B../collectImages.sh && ./mkupdate.sh
打包成统一固件 update.img
```
## 修改源代码
在以上的命令中，加上 “showcommands”，会打印编译时使用的命令， -j 指定多核同步编译
### make update-api
```
sudo apt install lib32z1 lib32ncurses5
	修改java代码后,需要 
	make update-api
	make sdk
```
## 丢弃本地修改

```
repo forall -vc "git reset --hard"
repo sync
```

## 编译app到系统应用
###  原理

System app can be updated, in two ways:

Updated by the ota updating, this will change the app directly, and the app still exist on /system.

updated by using package install api, this need the new version apk have the same signature with the one on /system, and the new version will be installed on /data. That means, if the device is reset, the new version will lose.

Ota update is a normal way for system app updating.

# adb

### 基本命令

#### TCP/IP

```
$adb connect 192.168.31.158
$adb root
$adb shell "mount -o remount,rw /system"
$adb push `pwd`/app/release/app-release.apk /system/app/solar/app-release.apk
$adb push app/build/intermediates/cmake/debug/obj/armeabi-v7a/* /system/lib/
$adb shell chmod 0644 /system/app/solar/app-release.apk
$adb shell am start -n cn.tsnail.europa.gpio/cn.tsnail.europa.gpio.MainActivity

adb install C:/work/example.apk

adb uninstall <app package name>

adb shell am set-debug-app -w com.example.jishuxiaoheiwu.appdebugsample
```

#### USB

```
adb usb
adb unstall com.example.flutter_app
```



### 问题处理

```java
no permissions (user in plugdev group; are your udev rules wrong?);
```

```
 lsusb 
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 004: ID 138a:0090 Validity Sensors, Inc. VFS7500 Touch Fingerprint Sensor
Bus 001 Device 003: ID 04f2:b531 Chicony Electronics Co., Ltd Integrated Camera
Bus 001 Device 006: ID ***2717:ff08*** Xiaomi Inc. Redmi Note 3 (ADB Interface)
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

```

```
sudo vi /etc/udev/rules.d/51-android.rules
SUBSYSTEM=="usb", ATTR{idVendor}=="2717", ATTR{idProduct}=="ff08", MODE="0666", GROUP="plugdev"
```

```
sudo udevadm control --reload-rules
```

```
adb devices
If it still doesn't work, unplug/replug the device.
```



### logcat

`adb logcat`
$ adb shell
```
# logcat

adb logcat ActivityManager:I MyApp:D *:S
```