ESP32
====

## 命令行

[参考地址](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/get-started/index.html#get-started-connect)

### 安装

```sh
sudo apt install python-is-python3
cd /TOOLS/SDK/esp32
git clone --recursive https://github.com/espressif/esp-idf.git
cd esp-idf
export IDF_TOOLS_PATH=/TOOLS/SDK/esp32/espressif
./install.sh
```

安装ninja

```shell
sudo apt install ninja-build
```



如果git慢的话可以直接下载，下载地址在

```http
https://github.com/espressif/esp-idf/releases
```

执行完成后，tool-chain会下载到`/TOOLS/SDK/esp32/espressif`

增加 get_idf 的命令别名到`.profile`

```sh
alias get_idf='. /TOOLS/SDK/esp32/esp-idf/export.sh'
```

### hello_world

```sh
cd /TOOLS/SDK/esp32/esp-idf/examples/get-started/hello_world
idf.py set-target esp32
idf.py build
idf.py -p /dev/ttyUSB0 flash
idf.py -p /dev/ttyUSB0 monitor
```

## vscode

在安装vscode环境之前，要求已经安装了esp-idf 和 espressif

#### 安装esp-idf插件

> 安装插件`espressif.esp-idf-extension`

### 打开配置

按F1键，输入ESP-IDF:Configure ESP-IDF extension，打开配置页面，安装后似乎也会自动弹出配置页面

#### Verify ESP-IDF Tools

Add your ESP-IDF virtual environment python executable absolute path. Example: /.espressif/python_env/idf4.0_py3.8_env/Scripts/python

```
/TOOLS/SDK/esp32/espressif/python_env/idf4.1_py3.8_env/bin/python
```

Please provide absolute paths separated by (:). Using ~, $HOME or %HOME% is not supported.
Example: If executable path is /myToolFolder/bin/openocd then use /myToolFolder/bin:/anotherToolFolder/bin

Please specify the directories containing executable binaries for required ESP-IDF Tools (Make sure to also include CMake and Ninja-build):
**|** **xtensa-esp32-elf |** **xtensa-esp32s2-elf |** **esp32ulp-elf |** **esp32s2ulp-elf |** **openocd-esp32 |**

```
/TOOLS/SDK/esp32/esp-idf/components/esptool_py/esptool:/TOOLS/SDK/esp32/esp-idf/components/espcoredump:/TOOLS/SDK/esp32/esp-idf/components/partition_table:/TOOLS/SDK/esp32/espressif/tools/xtensa-esp32-elf/esp-2020r2-8.2.0/xtensa-esp32-elf/bin:/TOOLS/SDK/esp32/espressif/tools/xtensa-esp32s2-elf/esp-2020r2-8.2.0/xtensa-esp32s2-elf/bin:/TOOLS/SDK/esp32/espressif/tools/esp32ulp-elf/2.28.51-esp-20191205/esp32ulp-elf-binutils/bin:/TOOLS/SDK/esp32/espressif/tools/esp32s2ulp-elf/2.28.51-esp-20191205/esp32s2ulp-elf-binutils/bin:/TOOLS/SDK/esp32/espressif/tools/openocd-esp32/v0.10.0-esp32-20191114/openocd-esp32/bin:/TOOLS/SDK/esp32/espressif/python_env/idf4.1_py3.8_env/bin:  /TOOLS/SDK/esp32/esp-idf/tools:
```



#### Custom environment variables to be defined.

Replace any ${TOOL_PATH} with absolute path for each custom variable.
For example: **${TOOL_PATH}/openocd-esp32/share/openocd/scripts** should be replaced as /.espressif/tools/openocd-esp32/share/openocd/

OPENOCD_SCRIPTS

```sh
/TOOLS/SDK/esp32/espressif/tools/openocd-esp32/v0.10.0-esp32-20191114/openocd-esp32/share/openocd/scripts
```

#### 如果出现如下问题

```
The following Python requirements are not satisfied:
psutil>=5.5.1
pygdbmi<=0.9.0.2
typing>=3.6.6
xmlrunner>=1.7.7
Please follow the instructions found in the "Set up the tools" section of ESP-IDF Getting Started Guide
```

则可以如下处理

```sh
cat EOF> /TOOLS/SDK/esp32/esp-idf/requirements.txt
psutil>=5.5.1
pygdbmi<=0.9.0.2
typing>=3.6.6
xmlrunner>=1.7.7
EOF
```

然后重新install.sh

```
/TOOLS/SDK/esp32/esp-idf/install.sh
```

## 问题处理

#### mynvs.c:16:10: fatal error: nvs_flash.h: No such file or directory

不知道为何生成的`compile_command.json`中的include缺少一些头文件，可以通过如下方法加入到cmake中

```cmake
idf_component_register(
    SRCS "mynvs.c"
    INCLUDE_DIRS "${IDF_PATH}/components/nvs_flash/include;${IDF_PATH}/components/spi_flash/include"
)
```



#### F1->tasks: run build task 无法使用的解决办法

出现如下类似的错误，原有是

1. espressif idf 插件中的`${config:idf.pythonBinPath}`设置的是默认的python，不是虚拟的python`/TOOLS/SDK/esp32/espressif/python_env/idf4.1_py3.8_env/bin/python`,可以在插件的配置中修改为虚拟的python，或者直接修改`settings.json`
2. 未`source $HOME/.profile`，一些环境变量未生效。

```
python
Traceback (most recent call last):
  File "/TOOLS/SDK/esp32/esp-idf/tools/idf.py", line 775, in <module>
    main()
  File "/TOOLS/SDK/esp32/esp-idf/tools/idf.py", line 691, in main
    checks_output = check_environment()
  File "/TOOLS/SDK/esp32/esp-idf/tools/idf.py", line 64, in check_environment
    print_idf_version()
  File "/TOOLS/SDK/esp32/esp-idf/tools/idf.py", line 113, in print_idf_version
    version = idf_version()
  File "/TOOLS/SDK/esp32/esp-idf/tools/idf_py_actions/tools.py", line 53, in idf_version
    version = subprocess.check_output([
  File "/usr/lib/python3.8/subprocess.py", line 411, in check_output
    return run(*popenargs, stdout=PIPE, timeout=timeout, check=True,
  File "/usr/lib/python3.8/subprocess.py", line 489, in run
    with Popen(*popenargs, **kwargs) as process:
  File "/usr/lib/python3.8/subprocess.py", line 854, in __init__
    self._execute_child(args, executable, preexec_fn, close_fds,
  File "/usr/lib/python3.8/subprocess.py", line 1702, in _execute_child
    raise child_exception_type(errno_num, err_msg, err_filename)
FileNotFoundError: [Errno 2] No such file or directory: 'git'
```



默认的setting.json如下

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build - Build project",
            "type": "shell",
            "command": "${config:idf.pythonBinPath} ${config:idf.espIdfPath}/tools/idf.py build",
     ...
 }      
```

需要修改为如下(可能和环境变量有关)

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build - Build project",
            "type": "shell",
            "command": "source $HOME/.profile && source /TOOLS/SDK/esp32/esp-idf/export.sh && /TOOLS/SDK/esp32/espressif/python_env/idf4.1_py3.8_env/bin/python ${config:idf.espIdfPath}/tools/idf.py build",
 ...
 }
```



### template-app

F1->ESP-IDF:create project->template-app，项目建成后，会自动打开`ONBORADING`页面，按照上文描述配置参数后，即可进行编译。

点击工具栏上的`build-project`进行编译。

## 重要快捷键

| key    | 说明         |
| ------ | ------------ |
| ctrl+` | 打开控制台   |
| F1     | 插件选择命令 |
|        |              |

## 开发

## vscode+cmake

使用纯粹的vscode+cmake的方式，进行开发，花2天时间将环境配置好，但是编译的时候，由于缺少了一些宏变量，无法编译通过，暂时放弃。待后续通过其他方式编译好了后，在回头调整。



## arduino

> [arduino esp扩展](https://github.com/espressif/arduino-esp32)

使用ttgo官方推荐的arduino进行开发，通过各种配置后，最终也无法编译成功。。。。

## platformIO

[官网](https://docs.platformio.org/en/latest/)

使用PIO，大概话费3个小时编译成功。总结经验如下：

### 基本概念

PIO中对于一种平台包含三总定义：

1. platform

   描述软硬件平台，如`platform = espressif32`

2. board 

   开发版，如`board=esp32dev`

3. framework

   开发框架，如`framework = arduino`

### 插件安装

> 在vscode中直接搜索platformIO，在搜索的结构中选择安装即可。

### .platformio目录

默认platformio会在`$HOME/.platformio`下安装需要的平台、编译环境、框架文件等。为了防止`HOME`爆满，可以通过软件链接的方式将文件存储到其他目录



### 基本原理

原理不能深究，简单来说有如下几点

1. arduino 框架怎么回事

   `at platformio/packages/framework-arduinoespressif32/cores/esp32/main.cpp`

   ```c++
   #include "freertos/FreeRTOS.h"
   #include "freertos/task.h"
   #include "esp_task_wdt.h"
   #include "Arduino.h"
   
   TaskHandle_t loopTaskHandle = NULL;
   
   #if CONFIG_AUTOSTART_ARDUINO
   
   bool loopTaskWDTEnabled;
   
   void loopTask(void *pvParameters)
   {
       setup();
       for(;;) {
           if(loopTaskWDTEnabled){
               esp_task_wdt_reset();
           }
           loop();
       }
   }
   
   extern "C" void app_main()
   {
       loopTaskWDTEnabled = false;
       initArduino();
       xTaskCreateUniversal(loopTask, "loopTask", 8192, NULL, 1, &loopTaskHandle, CONFIG_ARDUINO_RUNNING_CORE);
   }
   
   #endif
   ```

   而在`platformio/packages/framework-arduinoespressif32/tools/sdk/sdkconfig`中定义了这个宏`CONFIG_AUTOSTART_ARDUINO`

   ```ini
   #
   # Arduino Configuration
   #
   CONFIG_ENABLE_ARDUINO_DEPENDS=y
   CONFIG_AUTOSTART_ARDUINO=y
   ```

2. 何为sdkconfig

   >Application developers can open a terminal-based project configuration menu with the `idf.py menuconfig` build target.After being updated, this configuration is saved inside `sdkconfig` file in the project root directory. Based on `sdkconfig`, application build targets will generate `sdkconfig.h` file in the build directory, and will make sdkconfig options available to the project build system and source files

3. 如何编译的？

   PIO 使用自己的`pio`命令进行编译，具体逻辑尚不明确。pio命令是一个python写的脚本

4. 关于platfromio.ini

   ```ini
   ; PlatformIO Project Configuration File
   ;
   ;   Build options: build flags, source filter
   ;   Upload options: custom upload port, speed and extra flags
   ;   Library options: dependencies, extra library storages
   ;   Advanced options: extra scripting
   ;
   ; Please visit documentation for the other options and examples
   ; https://docs.platformio.org/page/projectconf.html
   
   [env:esp32dev]
   platform = espressif32
   board = esp32dev
   framework = arduino
   #I add this 
   monitor_speed = 115200
   build_flags =
     -Os
     -DCORE_DEBUG_LEVEL=ARDUHAL_LOG_LEVEL_DEBUG
     -DUSER_SETUP_LOADED=1
     -DST7789_DRIVER=1
     -DTFT_WIDTH=135
     -DTFT_HEIGHT=240
     -DCGRAM_OFFSET=1
     -DTFT_MISO=-1
     -DTFT_MOSI=19
     -DTFT_SCLK=18
     -DTFT_CS=5
     -DTFT_DC=16
     -DTFT_RST=23
     -DTFT_BL=4
     -DTFT_BACKLIGHT_ON=1
     -DLOAD_GLCD=1
     -DLOAD_FONT2=1
     -DLOAD_FONT4=1
     -DLOAD_FONT6=1
     -DLOAD_FONT7=1
     -DLOAD_FONT8=1
     -DLOAD_GFXFF=1
     -DSMOOTH_FONT=1
     -DSPI_FREQUENCY=40000000
     -DSPI_READ_FREQUENCY=6000000
   lib_deps =
       TFT_eSPI
       Button2
   ```

   `build_flags`为编译的宏

   `lib_deps`为依赖库，通过pio插件查询后（[也可以通过在线查询地址查询](https://platformio.org/lib)）可以配置在这里，在进行编译的时候，会自动下载。下载完成后会放到项目根目录的/lib目录下。

   

