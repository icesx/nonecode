Arduino
=====

## avrdude 

> avrdue是一个avr平台的工具，用于编译hex文件和烧写

### 基本命令

1. 支持的isp

   ```sh
   avrdude  -c avrisp
   ```

2. 烧写

   ```
   sudo avrdude -pm2560 -cstk500v2 -P/dev/ttyACM0 -b115200 -D  -Uflash:w:/ICESX/workSpaceC/SmartCarPark/product/code/frontend/demo/eclipse/Blink/ATMEGA2560/Blink.hex
   ```

   

## on eclipse

1. 参考文献：http://playground.arduino.cc/Code/Eclipse#Arduino_core_library
2. 参考归参考具体的还是要自己理解的按照实地的情况调整。
3. 本人搭建的与文献中的不同点：
   1. 文献中是一个project，而本人搭建的是多project，将通用的文件放到了一个project；适合多项目协调开发
   2. 本文以Ubuntu18.04为工作环境
4. 准备环境
   1. ubuntu18.04、java8.x、eclipse4.4.2【luna】
   2. arduino uno 3
   3. 安装eclipse插件：AVR Eclipse http://avr-eclipse.sourceforge.net/wiki/index.php/The_AVR_Eclipse_Plugin

```	sh
sudo apt-get install avrdude binutils-avr gcc-avr avr-libc gdb-avr
```

5. install arduionIDE

### 独立项目模式
 1. 建立eclipse项目ArduinoAIO	

 2. C++project

 3. AVR Cross Target Application

 4. create source folder “src”，“arduino”，“SoftwareSerial”,"standard"	

 5. cp sourcecode from arduionIDE
    		

     1. cp ${arduinoIDE}/hardware/arduino/avr/cores/* ArduinoAIO/arduino/
        			

     2. cp ${arduinoIDE}/hardware/arduino/avr/libraries/SoftwareSerial/* ArduinoAIO/SoftwareSerial/
        			

     3. cp \${arduinoIDE}/hardware/arduino/avr/variants/standard/* ArduinoAIO/standard/
        		

     4. ArdunioAIO-properties-AVR-AVRDude-programmer-New：Configer Name ：ATMEGA328P_16MHZ;programmer-hardware:Atmel STK500 Version 2.x firmware -D -OK
        		`sudo avrdude -pm2560 -cstk500v2 -P/dev/ttyACM0 -b115200 -D  -Uflash:w:/ICESX/workSpaceC/SmartCarPark/product/code/frontend/demo/eclipse/Blink/ATMEGA2560/Blink.hex`
     

     5. ArdunioAIO-properties-C/C++Build-ManageConfiger-New：ATMMEGA328P_16MHZ-SetActive
6. ArdunioAIO-properties-C/C++Build-Settings-Addtion Tools in Toolchain-checkon Genner HEX file for Flash Memory
   7. ArdunioAIO-properties-C/C++Build-Settings-AVR Compiler
     ​			

       1. Directores:and 

          ```
          "\${workspace_loc:/${ProjName}/arduino}"
          
          "\${workspace_loc:/\${ProjName}/standrad}"
          
          "\${workspace_loc:/\${ProjName}/SoftwareSerial}"
          ```

          
     2. debugging：No debug info
        
   3. Optimization：Size Optimization；other Optimization:-ffunction-sections -fdata-sections
        
        4. language Standara：un check all
     8. ArdunioAIO-properties-C/C++Build-Settings-AVR C++ Compiler
        ​			
     
        1. a-c、same above
           ​
        2. language Standara：only check on  no not use exceptions

### 多项目模式
A、建立eclipse项目ArduinoLib
    		所有的配置和ArduionAIO相似，区别的有如下两点
    		I、建立项目的时候，选择AVR Cross target statis Libaray
    		II、不要创建 source folder “src”
    		III、不要执行：ArdunioAIO-properties-C/C++Build-Settings-Addtion Tools in Toolchain-checkon Genner HEX file for Flash Memory
    	B、建立eclipse项目ArduinoProject
    		所有的配置和ArduionAIO相似，区别的有如下区别
    		I、不要创建sourcefolder “arduino”，“SoftwareSerial”,"standard"
    		II、ArduinoProject-properties-project Reference：check on ArduinoLib

### 实例

Ardunio mega2560

1. 在uno上跑的程序不一定通过简单的修改环境变量可以在mega上跑【原因是宏，但是找到这个原因的过程是非常的漫长】
1. 找问题的过程如下：
  1. 在uno上可以跑的blink在mega上通过eclipse编译后已经无法运行，现象是led不闪动，或者非常暗的闪动。
  2. 首先怀疑是硬件电压的问题，通过测量确实，13号脚的输出电压只有1V在高电平的情况下。
  3. 重新买了一个七星虫的版本【贵了不少，等了三天】，主要是想验证是否有问题，拿回来后发现仍然有问题。
  4. 找了七星虫的客服也不回，没办法，智能自食其力了。
  5. 如今可以排除硬件的问题，应该是软件的问题——经过测试发现使用arduinoIDE编译的库可以刷上去，并且正常。
  6. 在测试了一下，发现用eclipse编译的库，程序控制13针脚闪动的时候，其实是11号针脚在闪动——通过面包板已经证实。
  7. 如此再需要确定是刷程序的问题，还是编译的问题——使用ardunioIDE编译的HEX用eclipse刷上去的时候，是可以正常工作的，说明是eclispse编译的问题。
  8. 那如此再通过检查了一下eclipse的相关配置，发现一切均正常。
  9. 在网上找子资料的时候，发现有人将arduinIDE的编译命令打印了出来，顿时茅塞顿开。


  10. 将arduinoIED中的显示详细打开，一模了然可以看到编译和刷写的命令，从中发现，在mega上多了一个宏：ARDUINO_AVR_MEGA2560【之前曾经加过一个ARDUINO_ARCH_AVR在做eth的时候】
  11. 果然将这个宏增加到SmartCarParkFront-Lib，上led的闪动正常了。
  12. 同时在avrdude的的的命令也按照arduinoIED的修改了一下。

     ​	
2. 但是后来不知道怎么的，又不行了，于是又花了一天半时间来定位问题：

  3. 经过对比发现确实arduinoIDE的编译命令和eclipse的有差别，但是不管怎样将eclipse的命令改成和arduino的一样，都不行，LED仍然是不闪。

  4. 将arduino编译的o文件，cp到eclipse中来，连接程a文件，发现可以用。所以怀疑是o文件编译的问题。

  5. 再次检查两边的编译命令，仍然没有突破，于是想到了排除法。

  6. 将eclipse中的o文件全部替换成arduino的，使用命令连接：avr-ar rcs  "libSmartCarParkFront-Lib.a" *.o。连接出来的a文件是可以用的，led可以闪动。

  7. 于是将eclipse编译的o文件一个一个替换进去，发现除了wiring_digital.o以外，其他的所有的替换到都是正常的。

  8. 于是使用arduino的命令在命令行编译wiring_digital.c，使用eclipse的源文件，编译出来的不能用，于是怀疑源代码的问题。

  9. 将eclipse中的源代码和arduino安装目录的源代码进行对比，发现arduino.h文件中有一句话不同。

     不知道为何，arduino.h文件中有这样一句话：

    #include "../variants/leonardo/pins_arduino.h"；
    于是修改为#include <pins_arduino.h>.
  8. 一切正常了，而且ARDUINO_AVR_MEGA2560删除也没有问题。注：一定要将variants/mega/目录增加到编译的的Directors中。
	3. 关于打印，可能出现打印乱码的问题，修改的办法是增加宏：F_CPU=16000000L，默认好像是F_CPU=16000000UL，这个在mega上好似有问题
	4. avrdude: ser_open(): can't open device "/dev/ttyACM0": Permission denied
	    sudo chmod a+rw /dev/ttyACM0

6、我自己的环境
	A、SmartCarParkFront-Lib：

## On Vscode

