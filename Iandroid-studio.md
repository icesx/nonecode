### install
​	在ubuntu 64位上，需要安装如下几个包，否则，无法build
​	【Error:org.gradle.process.internal.ExecException: A problem occurred starting process 'command aapt''】	
​	$ sudo apt-get install lib32stdc++6
​	$ sudo apt-get install lib32z1

### NAK
0. sudo apt install gcc-aarch64-linux-gnu,g++-aarch64-linux-gnu
1. 编译so
	使用eclipse[aarch64-linux-gnu-gcc]编译的so是否可以在android上使用呢? 可以
	需要使用arm的gcc,也就是上文的aarch64-linux-gnu-gcc-8,
2. jni
	编写java类
	
	```
		package com.example.i.gpiotb.jni;

	public class JniUtil {
	    public static native String getJniString();
	    public static native void writeStderr();
	}

	```
	生成.h文件
	javah package com.example.i.gpiotb.jni.JniUtil
	在eclipse中实现相关方法,编译的so文件cp到android-studio中的app/libs/arm64-v8a[64bit] 或者 armeabi-v7a[32bit]

3. cmake 包含外部so
在cmakelist.txt中增加如下内容
```
add_library( wiringPi# 名称
             SHARED
             IMPORTED )
set_target_properties( # Specifies the target library.
                       wiringPi #名称

                       # Specifies the parameter you want to define.
                       PROPERTIES IMPORTED_LOCATION

                       # Provides the path to the library you want to import.
                        ${PROJECT_SOURCE_DIR}/libs/${ANDROID_ABI}/libwiringPi.so
                        )
```