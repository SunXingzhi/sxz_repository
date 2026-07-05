与Linux上的Make相同, CMake也是一种项目构建工具, 可以帮助用户方便的组织好源代码之间的构建方式. 不过比Make好的是他的兼容性, 3端支持(WIndows, Linux, Mac), 因此使用CMake组织C/C++项目结构, 是现在C/C++开发较为常用的选择. 

与Make对应使用Makefile管理构建规则相同, CMake同样是使用一个脚本文件:CMakeList.txt来构建, 这个文件名是固定的. 对于一个小项目可能只需要一个CMakeList, 如果是大型项目, 那么可能会涉及到多CMakeList的内容,每个CMakelist作为一个模块加入到项目中，实现复用解耦，便于团队管理.
要明白具体CMake如何使用, 需要了解一下几个重要概念, 例如"从源代码到可执行程序"的过程.

# From Source Code To Excutable Program
这期间主要进行了:
1) 预编译, 宏替换, 头文件复制等;
2) 编译. 将源文件编译为汇编文件(.s);
3) 汇编, 把所有汇编文件(主程序, 以及第三方库等)变成动态库/静态库;
4) 链接, 将每个库(.a/.so)文件内容放到主程序需要的地方, 就形成了可执行程序;
这里不深究其具体实现, 因为知道这些就够学习CMake了.
# 基本概念
## Target
Target(目标)就是最终要生成的可执行文件或者静态/动态代码库, CMake就把这些作为构建的目标. 因此一个CMakeList一定会有一个目标, 否则这个构建脚本(CMakeList)就是没有意义的.

下面可以根据一个最简单的CMakeList.txt来入手.这个CMakelist文件只有一个目标，即最后的可执行文件，没有链接其他的库。
```python
cmake_minimum_required(VERSION 3.xx)

# Set the project name
set(CMAKE_PROJECT_NAME freertos_move_stm32_cubemx_hal)

# Assignment the project name to target
project(${CMAKE_PROJECT_NAME})
# add excuted file, or named target, the argument after target_name is target's source files. Other kind of method is to use `target_source(${TARGET} src_lists})`, which benefits is to organize the project directly.
add_excutable(${CMAKE_PROJECT_NAME} main.c)

# If users have third target(static or dymantic library), they need to add.

```
## 依赖


## 作用域

在使用`target_include_dictionary()`, `target_link_dictionary()`等函数时，CMAKE给出了在传参中，目标名称后面的依赖前的**作用域**使用，即：
``` C
add_library(MyLib mylib.c)
// 三种作用域PUBLIC, PRIVATE, INTERFACE
target_include_dictionary(MyLib 
							PUBLIC {CMAKE_PROJECT_DIR}/inc
							PRIVATE {CMAKE_PROJECT_DIR}/src
							INTERFACE {CMAKE_PROJECT_DIR}/api)
```

精准的一句话用于解释这个作用域是：

| 关键字       | 作用范围                 |
| --------- | -------------------- |
| PUBLIC    | 该目标和所有依赖它的目标都可见      |
| PRIVATE   | 仅对该目标可见，依赖它的目标不可见    |
| INTERFACE | 仅对依赖该目标的目标可见（但自身不可用） |

对于INTERFACE的概念可能有点难以理解，这里举个例子：

```CMake
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h)
target_sources(main PUBLIC ${sources})
```

## 常用func
### 
## 多层项目组织
多层项目我们在进行项目管理时，如果把所有文件，依赖关系，库，目标堆在一个文件，会出现：多人团队难以维护，逻辑复杂等问题。为此，我们可以拆分项目成一个个模块/插件，每个插件可以单独赋予一个额外的CMakelist，实现更加分明的逻辑结构。
举个例子，在STM32 Cubemx 配置 CMake 工程生成的代码中，他的CMakelist文件结构如下：
```
主 CMakeLists.txt（总控）
│
├── add_subdirectory(cmake/stm32cubemx)
│   │
│   ├── add_library(stm32cubemx INTERFACE)     ← 头文件 + 宏
│   ├── add_library(STM32_Drivers OBJECT)      ← HAL 驱动源码
│   └── target_sources(主目标, 应用源码)
│
├── add_subdirectory(cmake/FreeRTOS)
│   │
│   └── add_library(freertos_11_3_0 INTERFACE) ← FreeRTOS 头文件
│
└── target_link_libraries(可执行文件
        stm32cubemx        ← 拉入 CubeMX 的 include/宏
        freertos_11_3_0    ← 拉入 FreeRTOS 的 include
    )

```
主 CMakeLists 只负责"组装"，每个子模块只负责"自己的东西"，互不干扰。这就是分层的意义。
>Note:`add_subdirectory()`,该目录下必须有CMakelist才生效。