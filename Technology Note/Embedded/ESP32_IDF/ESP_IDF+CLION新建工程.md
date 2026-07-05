# 前提
安装好ESP_IDF, CLION ，同时CLION配置好ESP工具链，开发环境（详情看官方教程）

# ESP_IDF新建项目
找到想要建项目的目录，使用终端打开，进入ESP-IDF专用终端, 输入：
```
cd <target_dir>
idf.py create-project <project_name>
idf.py set-target <esp32_board_name:esp32s3>
```
