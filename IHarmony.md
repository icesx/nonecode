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

### 开发

1. 下载代码

   ```
   git clone https://gitee.com/hihopeorg/Neptune-HarmonyOS-IOT
   ```

2. 增加代码

   使用`build.py w800`会编译`/applications/sample/wifi-iot/app`下面的代码，该配置文件在

   `/build/lite/product`下面。

   所以代码要增加到app下，最终目录结构如下

   ```
   ├── app
   │   ├── aht20
   │   │   ├── BUILD.gn
   │   │   ├── src
   │   │   │   ├── aht20.c
   │   │   │   ├── aht20.h
   │   │   │   └── BUILD.gn
   │   │   └── test
   │   │       ├── aht20_test.c
   │   │       └── BUILD.gn
   │   ├── BUILD.gn
   │   ├── gardener
   │   │   ├── BUILD.gn
   │   │   └── watering.c
   │   ├── iothardware
   │   │   ├── BUILD.gn
   │   │   ├── gpio_example.c
   │   │   ├── lowpower_example.c
   │   │   └── traffic_light_example.c
   
   ```

   gardener为增加的代码

3. 修改`BUILD.gn`

   首先修改`app/BUILD.gn`

   ```
   lite_component("app") {
       features = [
           # "startup",
           #"iothardware:gpio_example",
           # "wifitest:wifi_test",
           #"iothardware:lowpower_example"
           #"uart_sample:uart_demo"
           "gardener:watering"
       ]
   }
   ```

   修改`app/gardener/BUILD.gn`

   ```
   static_library("watering") {
       sources = [
           "watering.c"
       ]
   
       include_dirs = [
           "//utils/native/lite/include",
           "//kernel/liteos_m/components/cmsis/2.0",
           "//base/iot_hardware/interfaces/kits/wifiiot_lite",
       ]
   }
   ```

   

## DevEco (vscode)

[官方地址](https://device.harmonyos.com/cn/docs/documentation/guide/service_introduction-0000001050166905)

