与Linux上的Make相同, CMake也是一种项目构建工具, 可以帮助用户方便的组织好源代码之间的构建方式. 不过比Make好的是他的兼容性, 3端支持(WIndows, Linux, Mac), 因此使用CMake组织C/C++项目结构, 是现在C/C++开发较为常用的选择. 

与Make对应使用Makefile管理构建规则相同, CMake同样是使用一个脚本文件:CMakeList.txt来构建, 这个文件名是固定的. 对于一个小项目可能只需要一个CMakeList, 如果是大型项目, 那么可能会涉及到多CMakeList的内容.
要明白具体CMake如何使用, 需要了解一下几个重要概念, 例如"从源代码到可执行程序"的过程.

# From Source Code To Excutable Program
这期间主要进行了:
1) 预编译, 宏替换, 头文件复制等;
2) 编译. 将源文件编译为汇编文件(.s);
3) 汇编, 把所有汇编文件(主程序, 以及第三方库等)变成动态库/静态库;
4) 链接, 将每个库(.a/.so)文件内容放到主程序需要的地方, 就形成了可执行程序;
这里不深究其具体实现, 因为知道这些就够学习CMake了.
# Target
Target(目标)就是最终要生成的可执行文件或者静态/动态代码库, CMake就把这些作为构建的目标. 因此一个CMakeList一定会有一个目标, 否则这个构建脚本(CMakeList)就是没有意义的.

下面可以根据一个最简单的CMakeList.txt来入手.

# 依赖


# 作用域



```cmake
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h)
target_sources(main PUBLIC ${sources})
```