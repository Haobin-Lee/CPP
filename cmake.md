# cmake

## 库

### 静态库

静态库相当于直接把代码插入到生成的可执行文件中，导致体积变大

### 动态库

动态库只在生成的可执行文件中生成“插桩”函数，可执行文件被加载时读取指定目录中的`.dll(windows)`/`.so(Linux)`文件，加载到内存中空闲位置，替换相应的“插桩”指向的地址为加载后的地址，这个过程成为重定向。函数就跳转到动态加载的地址去。

**动态库查找位置**
windows 查找动态库，首先查找可执行文件同目录，其次是环境变量`%PATH`。
Linux ELF格式可执行文件的RPATH，其次是/usr/lib。



## C++ 样例

参考：https://www.jianshu.com/p/aaa19816f7ad

```cmake
#cmake最低版本要求
cmake_minimum_required(VERSION 3.10.0)

#设置全局变量TARGET_NAME，值为test
set(TARGET_NAME test)

#项目名字为test，编程语言为C++
project(${TARGET_NAME} LANGUAGES CXX)

#将 CMAKE_CURRENT_SOURCE_DIR 和 CMAKE_CURRENT_BINARY_DIR 添加到每一个文件夹，但不会传递到子目录。
set(CMAKE_INCLUDE_CURRENT_DIR ON)

# 导入Qt组件
find_package(Qt5 COMPONENTS Widgets REQUIRED)

#对于Qt targets，自动处理uic、moc、rcc
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

#设置遵循的标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#包含通用的编译环境模块到顶层目录，从给出的file文件加载和运行cmake代码
include(${CMAKE_SOURCE_DIR}/cmake/base.cmake)

#下一级的编译目录，添加子目录到编译系统
add_subdirectory(arch)
add_subdirectory(cfg)
add_subdirectory(doc)

#定义uvpy为可执行程序输出目标，最终可得到uvpy.exe
add_executable(${TARGET_NAME}
	main.cpp
	test.cpp
)
#往uvpy可执行程序输出目标添加源文件
target_sources(${TARGET_NAME}
	PRIVATE
		test02.cpp
)

#定义一个动态库shell 输出目标
add_library(shell SHARED
	consoleclient.cpp
)
#往shell动态库添加源文件
target_sources(shell
	PRIVATE
		highlighter.cpp
)

#定义一个静态库db 输出目标
add_library(shell0 STATIC
	commonparameter.cpp
)

#从给出的源文件直接生成object文件
add_library(shell1 OBJECT test03.cpp)

#头文件查找目录,相当于gcc -I选项
target_include_directories(${TARGET_NAME}
	PRIVATE
	${CMAKE_SOURCE_DIR}
)

#add_executable或add_library中定义的输出目标指定链接选项可以通过target_link_libraries命令全部搞定
# 链接选项设置
target_link_libraries(test_elf
    PRIVATE
    -msoft-float 
    -static 
    -nostdlib
)

#设置链接的标准路径的库
target_link_libraries(${TARGET_NAME} PRIVATE gcc)

#设置链接的本project自己输出的目标库libshell0.a
target_link_libraries(${TARGET_NAME} PRIVATE kernel)

#设置链接非标准路径中别人编译好的库
target_link_libraries(${TARGET_NAME} 
PRIVATE 
$ENV{HOME}/lib/libtest.a
)


```



## 内置变量

`CMAKE_SOURCE_DIR`: 表示当前项目根目录

## 常用命令

#### add_executable

#### add_library

将指定的源文件生成链接文件，然后添加到工程中

```cmake
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            source1 [source2 ...])
```

\<name>：表示库文件的名字，该库文件会根据命令里列出的源文件来创建。

STATIC：表示生成静态库，编译时生成。

SHARED：表示生成共享（动态）库，运行时被加载。

MODULE：一种不会被链接到其它目标中的插件，但是可能会在运行时动态加载。

默认是STATIC。

例： 声明一个库目标文件，运行时加载，也可以用target_sources构成源文件。

```cmake
add_library(math SHARED "")
 
target_sources(math
  PRIVATE
    ${CMAKE_CURRENT_BINARY_DIR}/wrap_BLAS_LAPACK/CxxBLAS.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/wrap_BLAS_LAPACK/CxxLAPACK.cpp
  PUBLIC
    ${CMAKE_CURRENT_BINARY_DIR}/wrap_BLAS_LAPACK/CxxBLAS.hpp
    ${CMAKE_CURRENT_BINARY_DIR}/wrap_BLAS_LAPACK/CxxLAPACK.hpp
  )
```

#### target_sources

往source文件中追加源文件，

```cmake
add_executable(my_source "")
 
target_sources(my_source
PRIVATE
subSource.cpp)
```

源文件指定为`PRIVATE`，是因为源文件只是在构建库文件是使用，头文件指定为Public是因为构建和编译时都会使用。

**target_include_directories**

指定包含目录。

**target_link_libraries**

指定要链接的库。

声明构建目标后，用taget_开头的命令添加构建该目标所需的资源和属性。

添加的资源或属性的可见性范围：

+ **PRIVATE**：**只用于**该Taget的构建，**不用于**使用该Taget的其他对象
+ **INTERFACE**：**只用于**使用该Target的其他对象
+ **PUBLIC**：**既用于**该Target的构建，**也用于**使用该Target的其他对象
