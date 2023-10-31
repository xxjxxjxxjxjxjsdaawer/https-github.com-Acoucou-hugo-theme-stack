+++
author = "coucou"
title = "开发工具——Makefile&CMake"
date = "2023-08-01"
description = "开发工具专题之Makefile&CMake"
categories = [
    "开发工具"
]
tags = [
    "开发工具","Makefile&CMake"
]
+++

![](1.jpg)

## Makefile

### Makefile 规则格式

```makefile
目标…... : 依赖文件集合……
	命令 1      # 命令列表中的每条命令必须以 TAB 键开始，不能使用空格！
	命令 2
	……
例1：
main : main.o input.o calcu.o
	gcc -o main main.o input.o calcu.o
# 这条规则的目标是 main，main.o、input.o 和 calcu.o 是生成 main 的依赖文件

例2：
main: main.o input.o calcu.o
	gcc -o main main.o input.o calcu.o
main.o: main.c
	gcc -c main.c
input.o: input.c
	gcc -c input.c
calcu.o: calcu.c
	gcc -c calcu.c

clean:
 	rm *.o
 	rm main
# 首先更新第一条规则中的 main，第一条规则的目标成为默认目标，只要默认目标更新了那么就认为 Makefile 的工作。在第一次编译的时候由于 main 还不存在，因此第一条规则会执行，第一条规则依赖于文件 main.o、input.o 和 calcu.o 这个三个.o 文件，这三个.o 文件目前还都没有，因此必须先更新这三个文件。make 会查找以这三个.o 文件为目标的规则并执行。

例3：
main: main.o input.o calcu.o
	gcc -o main main.o input.o calcu.o
	
%.o : %.c         # 通配符
	gcc -c $<     # 自动化变量，依赖文件集合中的第一个文件，如果依赖文件是以模式(即“%”)定义的，那么“$<”就是符合模式的一系列				   的文件集合。
clean:
 	rm *.o
 	rm main
```

### Makefile 变量

```makefile
# Makefile 变量的使用
objects = main.o input.o  # objects为变量
objects += calcu.o        # 追加
main: $(objects)
	gcc -o main $(objects)
```

### Makefile 函数

```makefile
# 格式
$(函数名 参数集合)

# 1. 字符串替换
$(subst <from>,<to>,<text>)
例： $(subst zzk,ZZK,my name is zzk)    # 把字符串“my name is zzk”中的“zzk”替换为“ZZK”

# 2. 获取目录
$(dir <names…>)
例： $(dir </src/a.c>)   # 返回 “/src”

# 3. 提取文件名
$(notdir <names…>)

# 4. 完成循环
$(foreach <var>, <list>, <text>)     # 意思就是把参数<list>中的单词逐一取出来放到参数<var>中，然后再执行<text>所包含的表达式。每次<text>都会返回一个字符串，循环的过程中，<text>中所包含的每个字符串会以空格隔开，最后当整个循环结束时，<text>所返回的每个字符串所组成的整个字符串将会是函数 foreach 函数的返回值
```

## CMake

>源文件：**CMakeLists.txt**

### CMake常用命令

```cmake
# 0
message("Hello World!")   # 打印"Hello World"
project(HELLO)   # 设置工程名称为 HELLO

# 1. add_executable 命令用于添加一个可执行程序目标，并设置目标所需的源文件
例：  add_executable(hello 1.c 2.c 3.c)   # 生成可执行文件 hello

# 2. add_library 命令用于添加一个库文件目标，并设置目标所需的源文件
例： add_library(mylib STATIC 1.c 2.c 3.c)  # 生成静态库文件 libmylib.a
	add_library(mylib SHARED 1.c 2.c 3.c)  # 生成动态库文件 libmylib.so

# 3. add_subdirectory 命令告诉 cmake 去指定的目录中寻找源码并执行它
例： add_subdirectory(src)   # 告诉 cmake 去 src 目录下寻找 CMakeLists.txt

# 4. aux_source_directory 命令会查找目录中的所有源文件
例：aux_source_directory(src SRC_LIST)  # 查找 src 目录下的所有源文件
    message("${SRC_LIST}") # 打印 SRC_LIST 变量

# 5. include_directories 命令用于设置头文件的搜索路径
include_directories(include)   # 头文件路径include

# 6. link_directories 命令用于设置库文件的搜索路径
# 	 link_libraries 命令会将指定库文件添加到链接库列表 
        例： link_directories(lib)
            link_libraries(${PROJECT_SOURCE_DIR}/lib/libhello.so)
# 7. target_include_directories 命令为指定目标设置头文件搜索路径
#	 target_link_libraries 命令为指定目标设置链接库文件
        例：target_link_libraries(hello-world PUBLIC libhello.so)
		   target_include_directories(hello-world PUBLIC libhello.so)
注：include_directories()、link_libraries()是针对当前源码中的所有目标，并且还会向下传递（譬如通过
add_subdirectory 加载子源码时，也会将其传递给子源码）。在一个大的工程当中，这通常不规范、有时还
会编译出现错误、混乱，所以我们应尽量使用 target_include_directories()和 target_link_libraries()，保持整个
工程的目录清晰。

# 8. set 命令用于设置变量
例： set(SRC_LIST 1.c 2.c 3.c 4.c 5.c) #设置变量 VAR1=Hello
	message(${VAR1})
```

### CMake常用变量

```cmake
1. $(PROJECT_SOURCE_DIR)   # 工程顶层目录，也就是顶层 CMakeLists.txt 源码所在目录
2. $(PROJECT_BINARY_DIR)   # 工 程 BINARY_DIR ， 也 就 是 顶 层 CMakeLists.txt 源码的BINARY_DIR
3. CMAKE_CURRENT_SOURCE_DIR   # 当前源码所在路径 
4. CMAKE_CURRENT_BINARY_DIR   # 当前源码的 BINARY_DIR

5. EXECUTABLE_OUTPUT_PATH	# 可执行程序的输出路径
6. LIBRARY_OUTPUT_PATH      # 库文件的输出路径
例： # 设置可执行文件和库文件输出路径
	set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
	set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
```

### CMake条件语句

```cmake
if(expression)
 	# then section.
 	command1(args ...)
 	command2(args ...)
	...
elseif(expression2)
 	# elseif section.
 	command1(args ...)
	command2(args ...)
	...
else(expression)
 	# else section.
 	command1(args ...)
	command2(args ...)
```

### CMake循环语句

```cmake
# foreach循环
foreach(loop_var arg1 arg2 ...)
 	command1(args ...)
 	command2(args ...)
	...
	endforeach(loop_var)
	
例：  # foreach 循环测试
set(my_list hello world china)
foreach(loop_var ${my_list})
 	message("${loop_var}")
endforeach()

foreach(loop_var RANGE 4)	# RANGE 关键字表示范围
 	message("${loop_var}")
endforeach()

set(my_list A B C D)
foreach(loop_var IN LISTS my_list)		# IN 关键字，循环列表
	message("${loop_var}")
endforeach()

# while...break 测试
set(loop_var 10)
while(loop_var GREATER 0) #loop_var>0 时 执行循环体
 	message("${loop_var}")
 	if(loop_var LESS 6) #当 loop_var 小于 6 时
 		message("break")
 		break() #跳出循环
 	endif()
 	math(EXPR loop_var "${loop_var} - 1")#loop_var--
endwhile()
```

### CMake函数

>通过 function()定义的函数它的使用范围是**全局**的，并不局限于当前 源码、可以在其子源码或者父源码中被使用

```cmake
# function 函数
function(<name> [arg1 [arg2 [arg3 ...]]])
 	command1(args ...)
 	command2(args ...)
 	...
endfunction(<name>)

例：# 函数名: xyz
function(xyz arg1 arg2)
 	message("${arg1} ${arg2}")
endfunction()
xyz(Hello World)	# 调用函数

# 函数名: xyz
function(xyz arg1 arg2)
 	message("ARGC: ${ARGC}")  # 传参数量
 	message("ARGV: ${ARGV}")  # 传参列表
 	message("ARGV0: ${ARGV0}")  # 第一个参数
 	# 循环打印出各个参数
 	set(i 0)
 	foreach(loop ${ARGV})
 		message("arg${i}: " ${loop})
 		math(EXPR i "${i} + 1")
 	endforeach()
endfunction()
# 调用函数
xyz(A B C D E F G)
```

### CMake宏定义

```cmake
macro(abc arg1 arg2)	# macro 宏
 	if(DEFINED ARGC)
 		message(true)
 	else()
 		message(false)
 	endif()
endmacro()

function(xyz arg1 arg2)		# function 函数
 	if(DEFINED ARGC)
 		message(true)
 	else()
 		message(false)
 	endif()
endfunction()

abc(A B C D)	# 调用宏
xyz(A B C D)	# 调用函数

# false
# true
```

### CMake交叉编译

```cmake
# CMakeLists.txt  配置 ARM 交叉编译
cmake_minimum_required(VERSION 3.16.3)

set(CMAKE_SYSTEM_NAME Linux) #设置目标系统名字
set(CMAKE_SYSTEM_PROCESSOR arm) #设置目标处理器架构

# 指定编译器的 sysroot 路径
set(TOOLCHAIN_DIR /usr/local/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf)
set(CMAKE_SYSROOT ${TOOLCHAIN_DIR})
# 指定交叉编译器 arm-linux-gcc 和 arm-linux-g++
set(CMAKE_C_COMPILER ${TOOLCHAIN_DIR}/bin/arm-linux-gnueabihf-gcc)
set(CMAKE_CXX_COMPILER ${TOOLCHAIN_DIR}/bin/arm-linux-gnueabihf-g++)

# 为编译器添加编译选项
set(CMAKE_C_FLAGS "-march=armv7ve -mfpu=neon -mfloat-abi=hard -mcpu=cortex-a7")
set(CMAKE_CXX_FLAGS "-march=armv7ve -mfpu=neon -mfloat-abi=hard -mcpu=cortex-a7")
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
#################################
# end
##################################

project(HELLO) #设置工程名称
add_executable(main main.c)


# 编译选项 详见：https://blog.csdn.net/u013836909/article/details/107770465
-march=name  # 这指定了目标ARM体系结构的名称
```

