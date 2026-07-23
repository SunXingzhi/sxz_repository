# U-Boot

## 移植U-Boot 

移植U-Boot需要先理解整个U-Boot的工程结构以及工作流程.

### U-Boot的工程结构

| 目录               | 作用                                                         |
| :----------------- | :----------------------------------------------------------- |
| `arch/`            | 架构相关代码，如 `arch/arm/cpu/armv7/` (CPU 初始化)、`arch/arm/dts/` (设备树源文件) |
| `board/`           | **板级支持包**，每个板子的特定初始化代码，如 `board/freescale/mx6ull_alientek_emmc/` |
| `configs/`         | **板级 defconfig 文件**，控制 Kconfig 选项，如 `mx6ull_alientek_emmc_defconfig` |
| `include/configs/` | **板级头文件**，传统宏定义（环境变量、内存地址、外设基址等），如 `mx6ull_alientek_emmc.h` |
| `drivers/`         | 外设驱动：`mmc/`, `net/`, `i2c/`, `gpio/` 等                 |
| `common/`          | 通用命令（`cmd_*.c`）、启动流程、环境变量处理                |
| `env/`             | 环境存储后端实现：MMC、NAND、flash、ext4 等                  |
| `fs/`              | 文件系统支持：fat、ext4、ubifs                               |
| `net/`             | 网络协议栈：TFTP、DHCP、ping、NFS                            |
| `tools/`           | 主机工具：`mkimage` 等                                       |
| `Makefile`         | 顶层编译入口                                                 |
| `Kconfig`          | 菜单化配置入口                                               |

其中移植时的终点在`dts`,`board/xx`,`include/configs/xx.h`, `drivers`等等.

---

### U-Boot 启动流程(以I.MX6ULL ALPHA -ALIENTEK EMMC 512MB 为例)

分为芯片初始化和扳机初始化,即`Arch`层和`Board`层

#### 1. 处理器上电与 Boot ROM

i.MX6ULL 上电后，首先执行片内 Boot ROM 固化的启动代码。它会根据 BOOT_MODE 引脚选择启动设备（SD 卡、eMMC、NAND 等）。
对于从 SD 卡启动，ROM 会读取卡上固定偏移（通常 1 KB）的映像，该映像必须是带 **IVT（Image Vector Table）** 和 **DCD（Device Configuration Data）** 的 `u-boot.imx` 格式。详情可以看`./Embedded/IMX6ULL/2.启动方式`一节.

- **DCD 表**：在加载映像到内存之前，ROM 会根据 DCD 表的描述，初始化 DDR 控制器和时钟，使 DDR 内存立即可用。
- **IVT**：指向入口点地址，ROM 将完整映像加载到 DDR 后，直接跳转到入口（即 U-Boot 的 `_start`）。

> 由于 DDR 在进入 U-Boot 前已就绪，i.MX6ULL 通常**不需要 SPL**，U-Boot 自身直接在 DDR 中运行。

------

#### 2. 汇编入口与 C 环境建立（`_start`）

入口代码位于 `arch/arm/cpu/armv7/start.S`，依次完成：

- 设置异常向量表。
- 切换处理器到 SVC 模式。
- 关闭 MMU 和 DCache，使能 ICache。
- 初始化栈指针（早期栈通常设在内部 RAM 中）。
- 跳转到 C 函数 `board_init_f()`，参数为 0。

------

#### 3. 早期初始化 `board_init_f`（运行于 DRAM，但无重定位）

该函数位于 `common/board_f.c`，通过一个**初始化序列数组 `init_sequence_f`** 逐步执行：

- **串口初始化** (`init_baud_rate`, `serial_init`)：建立控制台输出，方便调试。
- **时钟与定时器初始化**：提供 `get_timer()` 等基本函数。
- **DRAM 信息采集** (`dram_init`)：获取内存大小（通常从 DCD 配置或硬件探测）。
- **计算重定位目标地址**：U-Boot 会将自身搬移到 DRAM 顶部，留出空间给内核等镜像。
- **设置全局数据结构**（`global_data`）的 `gd->relocaddr`、`gd->ram_top` 等。
- **初始化内存分配区**（`malloc` 池）并设置环境变量缓冲区。
- **串口输出信息**：打印 CPU 频率、内存大小等（你看到的 `CPU: ... DRAM: 512 MiB` 即在此阶段输出）。
- 最后调用 `relocate_code()` 进行代码重定位。

------

#### 4. 重定位 `relocate_code`

- 将 U-Boot 自身的 **代码段、只读数据、数据段** 全部复制到 `gd->relocaddr` 指向的 DRAM 顶部区域。
- 修正 `.rel.dyn` 中的重定位表项，使所有绝对地址引用指向新位置。
- 跳转到重定位后的 C 函数 `board_init_r()`。

重定位后，U-Boot 的全局变量可以被正确读写，以前在 flash/小容量 RAM 中无法写入的 BSS 段也完全可用。

------

#### 5. 后期初始化 `board_init_r`（运行于重定位后）

函数位于 `common/board_r.c`，通过**初始化序列 `init_sequence_r`** 执行剩余所有关键初始化，**顺序非常重要**：

1. **使能 Cache、MMU**（如果需要）。
2. **中断初始化**。
3. **控制台完全就绪**（`initr_console`），之后能用 `printf`。
4. **环境变量加载** (`initr_env`) —— **这就是你问题的关键环节**：
   - 调用 `env_relocate()`，根据配置从存储设备（这里是 MMC）读取环境数据。
   - 如果 CRC 校验失败或读不到，则回退到**编译时内嵌的默认环境**（`default_environment`）。
   - 你的板子因为 `CONFIG_SYS_MMC_ENV_DEV=1`（指向 eMMC），在 SD 卡上读环境失败，所以每次都只能用默认环境。如果默认环境里没有 `ipaddr` 或有残缺的 `ipaddr`（带空格），就会出现 `ipaddr not set` 但 `printenv` 看似正常的现象。
5. **外设初始化**（按顺序）：
   - `initr_mmc`：MMC/SD 子系统初始化，**之后才能用 `mmc read/write`**。
   - `initr_net`：网卡初始化（FEC 驱动），**依赖于环境变量中的 `ethaddr`**。若没有 `ethaddr`，部分驱动会拒绝初始化，导致 `No ethernet found`。
   - `initr_usb`、`initr_video` 等。
6. **看门狗、PHY 等**。
7. **打印启动信息**：你看到的 `Board: MX6ULL ALIENTEK EMMC`、`Net: FEC1` 等信息均在各自驱动初始化时打印。
8. **将环境变量和全局数据同步**，执行 `run_preboot_environment_command` 之类的脚本。

------

#### 6. 主循环 `main_loop`

所有初始化完成后，U-Boot 进入 `common/main.c` 的 `main_loop()`：

- 设置 `bootdelay` 倒计时，若被用户打断则进入 **命令行交互模式**（你用的 `=>` 提示符）。

- 若无人打断，则自动执行 `bootcmd` 环境变量定义的命令。典型 `bootcmd` 会：

  text

  ```
  run findfdt; mmc dev ${mmcdev}; mmc dev ${mmcdev};
  if mmc rescan; then
      if run loadbootscript; then run bootscript;
      else if run loadimage; then run mmcboot;
      else run netboot; fi;
      fi;
  else run netboot; fi
  ```

  

  - 加载设备树 (`loadfdt`) 和内核镜像 (`loadimage`) 到内存，再通过 `bootz` 跳转到内核入口，交出控制权。

在uboot成功启动后, 实际上就会按照这个主循环的内容开始运行, 如果用户在扫描周期内没有输入, 则会进入上面的逻辑运行`Loadbootscript/loadImage/netboot`三种情况, 而这三种情况到底会运行那一个, 实际上就是根据两个重要的环境变量`bootcommand`和`bootargs`决定的, 这里可以接着展开.



---

## 网络配置

## U-Boot编译流程分析(Makefile+menuconfig图形化配置)





---

## 常见问题

#### 下载移植好的uboot程序(包括LCD驱动, ENET1,2 驱动), 到SD卡后(官方的`imxdownload`程序) , 配置serverip与本机ip后, 尝试ping, 出现: abort, 并展示出异常位置:

这种其实是交叉编译工具链版本过高, 应该以教程中提供的交叉编译工具链版本(4.9 linaro)为准.

