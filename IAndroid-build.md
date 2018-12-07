about android build
##############

##prepare
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



##tinkerboard
###编译u-boot

```
apt install bc
cd [source tree]/u-boot
make rk3288_defconfig
```
###编译kernel
```
sudo apt install lzop gcc-multilib
A.cd [source tree]/kernel
B.make rockchip_defconfig
C.make rk3288-miniarm.img
```
###编译android
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
###统计固件
打包成统一固件 update.img
在 Windows 下打包统一固件 update.img 很简单，按上一步骤将文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。

update.img 方便固件的发布，供终端用户升级系统使用。一般开发时使用分区映像比较方便。
