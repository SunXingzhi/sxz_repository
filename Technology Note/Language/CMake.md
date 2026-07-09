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
### 系统基本函数

| API                      | 功能                                         | Example                                                                                                                                                                                             |
| ------------------------ | ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| cmake_minimum_required() | 规定CMake最低要求版本                              | cmake_minimum_required(VERSION 3.22)                                                                                                                                                                |
| message()                | CMake打印输出函数，后面直接加字符串，支持引用字符串变量。支持传入变量类型/模式 | <br>message("普通信息")                    # 普通输出<br>message(STATUS "状态信息")             # 带 -- 前缀，一般信息<br>message(WARNING "警告")                # 黄色警告<br>message(FATAL_ERROR "致命错误")        # 红色错误，停止配置 |
|                          |                                            |                                                                                                                                                                                                     |
|                          |                                            |                                                                                                                                                                                                     |
|                          |                                            |                                                                                                                                                                                                     |

### 路径管理相关函数

| API                | 功能                                                                                                                                                                                                                                                                         | Example                             |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| file(GLOB)         | 选择将目录路径字符串列表作为一个变量的值。<br>同时支持通配符等表达式。<br>需要注意的是他并不会在每次CMakelist更新后自动检测文件的更改。因此每次都要清楚缓存并重新生成。并且默认也不会检测文件列表中的每个文件是否改变了，如果想要检测，需要在文件列表路径之后添加 CONFIGURE_DEPENDS 参数<br>> NOTE: 需要注意的是，官方不推荐用 `file(GLOB)` 收集源文件，因为：<br>- 无法精确知道哪些文件被添加/删除了<br>- 可能导致意外的重新编译，**手动列出源文件**是官方推荐的做法 |                                     |
| add_subdirectory() | 设置子CMakelists.txt搜索路径                                                                                                                                                                                                                                                      | add_subdirectory(cmake/stm32cubemx) |
|                    |                                                                                                                                                                                                                                                                            |                                     |

### 目标相关函数

| API                                   | 功能                                                                                                | Example                                                                                                                                                                                             |
| ------------------------------------- | ------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| add_excute(${target_name} \<source>)  | 添加可执行程序目标                                                                                         | )                                                                                                                                                                                                   |
| add_library(${target_name} \<source>) | CMake打印输出函数，后面直接加字符串，支持引用字符串变量。支持传入变量类型/模式                                                        | <br>message("普通信息")                    # 普通输出<br>message(STATUS "状态信息")             # 带 -- 前缀，一般信息<br>message(WARNING "警告")                # 黄色警告<br>message(FATAL_ERROR "致命错误")        # 红色错误，停止配置 |
| target_source()                       | 为目标增加源文件                                                                                          | target_sources(my_app PRIVATE<br>    Core/Src/main.c<br>    Core/Src/gpio.c<br>    Core/Src/stm32f1xx_it.c<br>)                                                                                     |
| target_include_directories()          | 设置头文件搜索路径                                                                                         | target_include_directories(my_app PRIVATE<br>    Core/Inc<br>    Drivers/CMSIS/Include<br>)                                                                                                         |
| target_link_directories()             | 设置库文件搜索路径                                                                                         |                                                                                                                                                                                                     |
| target_compile_definitions()          | 添加编译器定义宏。如果是嵌入式（STM32)，可以是使用的库宏（USE_HAL_DRIVER or USE_STANDARD_FIWAXX);<br>以及芯片名称（STM32F103XB)等等。 | target_compile_definitions(my_app PRIVATE<br>    USE_HAL_DRIVER<br>    STM32F103xB<br>    DEBUG=1<br>)                                                                                              |
|                                       |                                                                                                   |                                                                                                                                                                                                     |
## 常用系统内置变量
定义CMake/第三方库的CMakelist要求的变量，可以方便的配置项目。CMake中常见的内置变量有，C语言标准，项目名称， 项目目录等等。具体常用变量如下：


| Variable                      | Func                                                                  | Example                                                |
| ----------------------------- | --------------------------------------------------------------------- | ------------------------------------------------------ |
| CMAKE_C_STANDARD              | 标明C标准                                                                 | set(CMAKE_C_STANDARD 11)                               |
| CMAKE_C_STANDARD_REQUIRED     | 设置变量表示是否要检查C标准版本                                                      | set(CMAKE_C_STANDARD_REQUIRED ON)                      |
| CMAKE_C_EXTENSIONS            | 设置是否要开启GCC 拓展的变量。嵌入式中常用的GCC拓展有typeof(), 零长度数组等。                       | set(CMAKE_C_EXTENSIONS ON)                             |
| CMAKE_BUILD_TYPE              | CMake构建类型，有"Debug","Release"等                                         | set(CMAKE_BUILD_TYPE "Debug")                          |
| CMAKE_PROJECT_NAME            | 约定俗称的项目变量。设置变量后可以project()函数传参，明明项目                                   | set(CMAKE_PROJECT_NAME freertos_move_stm32_cubemx_hal) |
| CMAKE_EXPORT_COMPILE_COMMANDS | 是否导出`compile_commands.json`文件。该文件常用于IDE语法跳转，高亮，补全等功能，如vscode的CLangd插件 | set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)                |
|                               |                                                                       |                                                        |

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
>Note:`add_subdirectory()`,该目录下必须有`CMakelists.txt`才生效。

主目录的CMakelists.txt中一定有的是