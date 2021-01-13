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

```
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

### 目录结构

