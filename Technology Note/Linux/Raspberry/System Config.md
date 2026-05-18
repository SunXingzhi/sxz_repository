# 简明教程
[【新手向】轻松上手树莓派4B（无痛开机&初始软件配置）_树莓派4b配置-CSDN博客](https://blog.csdn.net/m0_50679156/article/details/114483314)

# 需要安装的软件(Windows)

SD card formatter-SD格式化软件

raspberry pi imager-官方树莓派系统烧录程序

advanced ip scanner - 扫描局域网内所有IP及MAC地址

putty-用于远程登陆树莓派系统

puttygen-用于生成SSH登录用的密钥对（公钥+私钥组合）

VNC-用于显示树莓派系统的图形界面（可选） | 下载链接：[Download VNC Viewer by RealVNC®](https://www.realvnc.com/en/connect/download/viewer/?lai_vid=nXvAQ8R82f0Bl&lai_sr=5-9&lai_sl=l)

# 远程连接树莓派系统（局域网）

1.使用puttygen生成RSA密钥对（过程有点慢），保存公钥（public_key/authorized_key)值以及私钥（private_key)文件（.ppk)格式（名称自拟）

2.读卡器插入电脑，插入SD卡，打开pi imager软件，选择要烧录的设备、系统和烧录的SD盘名（一般是bootfs），然后 看下文具体的配置层选项。

3.点击烧录后，等待完成。

4.拔出SD卡，对树莓派通电前（可以使用电脑USB接口/电源插头供电）插入SD卡，然后通电。等待2分钟左右，如果手机热点上出现“raspberry”，证明系统已经成功运行

5.打开putty，创建一个session，Host_name就是ip，端口port默认为22。接下来需要导入puttygen生成保存然后默认用户名为q1325（可以更改）

# 需要注意几个问题

树莓派的WIFI连接问题

在使用官方的Raspberry Pi Imager程序进行烧录系统时，需要更改其默认配置再进行烧录。进入默认设置快捷键：ctrl+shift+x 。有几个必须设置的选项：

通用->勾选 设置用户名 | 设置用户名及密码 （这一步| 配置WLAN（及手机热点或家庭WIFI | 设置WLAN区域为 CN （中国）本地化设置不需要配置。

服务->勾选开启SSH登录 | 使用公钥进行登录，这里就导入puttygen生成的publickey。

树莓派不可以连接校园网，有墙；

使用putty进行远程连接时需要提供树莓派的IP。但是连接手机热点时，如果是安卓系统（version>Android12)，会有安全限制，即连接的设备不会显示ip，这个时候有几种方法获取raspberry的ip：

手机端

暂不支持，如果有手机连接热点后可以直接显示树莓派设备IP，请列出手机品牌；

PC端

1）将电脑也同时连入手机热点，然后打开命令行输入：arp -a后获得ip-MAC地址映射表，找到与手机热点连接设备中的raspberry的MAC地址一致的项对应的IP即可。

2）下载Advanced IP scanner 软件，该软件可以扫描局域网内的所有ip（图形化界面，同时可以访问其文件，推荐）。

python系统环境

树莓派系统本身限制直接使用pip 在系统默认python路径下安装依赖，防止破坏系统稳定性与安全。推荐使用虚拟环境进行配置python依赖：

`# 1. 创建项目目录`

`mkdir ~/xbox-robot && cd ~/xbox-robot`

`# 2. 创建虚拟环境`

`python3 -m venv venv`

`# 3. 激活`

`source venv/bin/activate`

`# 4. 安装依赖`

`pip install --upgrade pip`

`pip install inputs pyserial`

`# 5. 创建脚本`

`nano xbox_control.py`

编辑器环境

自带nano编辑器和vi编辑器，不好用，需要下载linux配套的vim编辑器：

sudo apt-get install vim

重启系统相关命令

| 常用命令                 | 含义   |
| -------------------- | ---- |
| sudo reboot          | 立即重启 |
| sudo poweroff        | 关闭电源 |
| sudo shutdown -h now | 立即关机 |
| sudo shutdown -h +2  | 延迟关机 |
|                      |      |
