# IMX6ULL EMMC (正点原子ALPHA 2.0开发板) qemu仿真

## 需要注意的点

### 编译busybox

编译时可以指定一个目录, 这样输出的`/sbin, /usr, linuxc`等文件就会放到那个目录中, 更加整洁;

### 创建其他目录

```bash
mkdir dev etc lib mnt proc root sys tmp usr
```

### 拷贝交叉编译库

这里注意拷贝的内容一定是编译busybox所使用的编译器版本

### ⭐增加启动相关配置文件(`rcS`,`inittab`,`fstab`)

