cmake
=====
> cmake 通过 CMakeLists.txt来进行编译的设定。

### 多目录情况

在一个项目有多个目录的情况下，如果一个目录A下有CmakeLists.txt那么这个目录就可以通过`add_subdirectory(A)`添加到编译过程中。在`A/CmakeLists.txt`文件中编写A目录下的编译过程

### 头文件

在CmakeLists.txt中可以设置头文件的查找路径代码如下

```cmake
include_directories(${CMAKE_CURRENT_LIST_DIR}/include)
```

include_directories中可以包含多个目录用空格分开如include_directories

```cmake
include_directories(
	A
	B
	C
)
```



### 变量设置

CMakeLists.txt中可以设置变量，该变量是可以在subdirectory中查看到的，如

```cmake
set(IVOKE 1)
```

### 逻辑判断

CMakeLists.txt中提供了if-else的判断逻辑，一般用于对一些变量设置的时候的判断，如

```cmake
if (${IVOKE} EQUAL 1)
    message("c++ ivoke c")
    set(SHARD_PATH lib_c_shard)
    set(LIB c_shard)
    set(EXE_PATH cpp_i_c)
else()
    message("c ivoke c++")
    set(SHARD_PATH lib_cpp_shard)
    set(LIB cpp_shard)
    set(EXE_PATH c_i_cpp)
endif()
```



### 循环

CMakeLists.txt中通过循环可以处理一些重复性的工作，如

```cmake
set(subs "array;callback;integer;method;string;struct")
foreach(sub IN LISTS subs)
    message("config subdirectory::: ${sub}")
    add_subdirectory("src/${sub}")
    link_directories(${PROJECT_BINARY_DIR}/src/${sub})
    target_link_libraries(${PROJECT_NAME} ${sub})
endforeach (sub)
```

### 内置变量

| 变量                                            | 说明                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| CMAKE_VERBOSE_MAKEFILE                          | 打开详细的make过程日志`set( CMAKE_VERBOSE_MAKEFILE on )`     |
| CMAKE_BINARY_DIR,PROJECT_BINARY_DIR,_BINARY_DIR | 这三个变量内容一致，如果是内部编译，就指的是工程的顶级目录，如果是外部编译，指的就是工程编译发生的目录 |
| CMAKE_SOURCE_DIR,PROJECT_SOURCE_DIR,_SOURCE_DIR | 这三个变量内容一致，都指的是工程的顶级目录                   |
| CMAKE_CURRENT_BINARY_DIR                        | 外部编译时，指的是target目录，内部编译时，指的是顶级目录     |
| CMAKE_CURRENT_SOURCE_DIR                        | CMakeList.txt所在的目录                                      |
| CMAKE_CURRENT_LIST_DIR                          | CMakeList.txt的完整路径                                      |
| CMAKE_CURRENT_LIST_LINE                         | 当前所在的行                                                 |
| CMAKE_MODULE_PATH                               | 如果工程复杂，可能需要编写一些cmake模块，这里通过SET指定这个变量 |
| LIBRARY_OUTPUT_DIR,BINARY_OUTPUT_DIR            | 库和可执行的最终存放目录                                     |



### 宏设置



### 常用方法

1. message

   ```cmake
   message("SHARD_PATH=${SHARD_PATH}")
   ```

2. set 

   设置变量，如

   ```cmake
   set( CMAKE_VERBOSE_MAKEFILE on ) 打开更详细的打印
   ```

   

3. cmake_minimum_required(VERSION 3.0.0)

4. project(ILC VERSION 0.1.0)

5. 设置编译选项

   ```cmake
   SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -m32")
   
   SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -m32")
   ```

6. 开启调试

   ```cmake
   set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} -g)
   ```

7. aux_source_directory

   ```cmake
   aux_source_directory(. DIR_LIB_SRCS)
   ```

   收集指定目录中所有源文件的名称，并将列表存储在提供的`DIR_LIB_SRCS`变量中。 该命令旨在供使用显式模板实例化的项目使用。

8. add_library

   ```cmake
   add_library(<name> [STATIC | SHARED | MODULE]
               [EXCLUDE_FROM_ALL]
               source1 [source2 ...])
   ```

   

   ```cmake
   add_library(struct ${DIR_LIB_SRCS})
   ```

   构建libstruct.so库，并添加到工程中去，使用变量`DIR_LIB_SRCS`中设置的源文件

9. add_executable

   ```cmake
   add_executable(< name> [WIN32] [MACOSX_BUNDLE]
   [EXCLUDE_FROM_ALL]
   source1 source2 … sourceN)
   ```

    使用给定的源文件，为工程引入一个可执行文件

10. target_link_libraries

    ```cmake
    target_link_libraries(<target> [item1] [item2] [...]
                          [[debug|optimized|general] <item>] ...)
    ```

    将目标文件`target`与库文件`item1-2..`进行链接。

11. link_directories

    ```cmake
    link_directories(
        ${PROJECT_BINARY_DIR}/lib_cpp_shard
        ${PROJECT_BINARY_DIR}/lib_c_shard
    )
    ```

    添加需要链接的库文件目录

12. 

### 编译选项

1. 设置C、C++的编译选项

   ```cmake
   add_compile_options(-Wall -Werror -Wstrict-prototypes -Wmissing-prototypes)
   ```

   ```cmake
   ADD_DEFINITIONS("-Wall -Werror -Wstrict-prototypes -Wmissing-prototypes)
   ```

2. 设置C或者C++编译选项

   ```cmake
   set(CMAKE_C_FLAGS "-Wall -Werror -Wstrict-prototypes -Wmissing-prototypes)
   ```

   

### 自定义模块



### 基本命令

`cmake . -LH`

## 使用例子

### arduino整合

