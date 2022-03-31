Harmony
===

## neptune开发板

### 编译与烧录

```sh
cat > build.sh <<EOF
export PATH=/TOOLS/SDK/csky/csky-elfabiv2-tools-x86_64-minilibc-20210423/bin:$PATH
python build.py w800
EOF
```



```sh
#先进入源码文件夹,可能目录和我不太一样，主要就是进入刚刚下载的文件夹内
cd Neptune-HarmonyOS-IOT
#编译命令
./build.sh
```

#### 烧录

crt打开串口/dev/ttyusb0
在电脑端按住键盘的‘ESC’，开发板按下‘RST’键，先松开‘RST’键再松开键盘的‘ESC’，当工具界面出现ccccc，即成功进入烧录模式

cccccccccccccccccc

点击软件的 Transfer ----> Send Xmodem

选中w800.img

### 问题

https://developer.huawei.com/consumer/cn/forum/topic/0203380024404140371



## DevEco (vscode)

[官方地址](https://device.harmonyos.com/cn/docs/documentation/guide/service_introduction-0000001050166905)

