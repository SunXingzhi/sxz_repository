# Makefile

## 依赖

依赖可以是一个文件, 多个文件, 一个目标, 一个伪目标, 多个目标, 以及所有的混合.



## 参数传递性

当有子目录Makefile加入根目录Makefile工程时, 根目录makefile 会传递变量(也可看做构建参数)到子目录的Makefile中. 使用`export/unexport`明确哪些变量要传递, 哪些变量不传递. 有两个变量在默认情况下是一定会传递的:`MAKEFLAGS`, `SHELL`, 其中`MAKEFLAGS`代表构建时的一些参数, 比如'@', 'V=1'(开启调试日志), '-s'(静默输出等等).



## Makefile中使用命令行(`Bash`为例)

在目标下方的语句中, 本质都是在命令行执行的内容. 如果要获取ming'ling'ha

### 自动化构建变量



## Make内置函数

### `filter $(a)`

keyi 

