# R2S-Armbian
## Armbian 的安装与基础配置
以下内容基于 Armbian Jammy CLI 版本，默认在 root 权限下操作。
安装 Armbian 的第一步就像 OpenWrt 一样，从 官方网站 下载对应的 img（本文使用基于 Ubuntu 的 Jammy），然后用你的工具烧录进 SD 卡。将 R2S 接入电源，指示灯应该闪烁，不会像 openwrt 一样表现为 sys 红灯闪烁直至系统启动完毕红灯常亮，而是 heartbeat 模式双闪。

初始设置需要将其 WAN 或 LAN 口接到另一台 已经启用 DHCP 的路由器上，此时需在主路由管理员后台上查看 armbian 内网地址后通过 ssh 进入进行初始化，初始用户名和密码分别为 root 和 1234。注意其此时不能作为一个开箱即用的路由器，也就是说不能直接接入电脑。

登入后有一个初始引导，跟着做就行了，没什么好说的。然后这就是一个标准的 Linux 系统了，基本可以直接遵循其他 Linux 的操作方法。但国内许多用户会到手换源，由于 Armbian 是基于其他发行版做的，所以换源不完全一样。其软件源有两个位置，均需要更改：

/etc/apt/sources.list 与你安装的版本所基于的发行版有关，如 Armbian Jammy 基于 Ubuntu Jammy，就换对应的源（如 [清华源](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu-ports)，注意 Ubuntu 使用 Ports 源）；
/etc/apt/sources.list.d/armbian.list 是 Armbian 专属软件源，也要换（仍然如 [清华源](https://mirrors.tuna.tsinghua.edu.cn/help/armbian)）。

## 使用系统内置的静态IP
使用SSH连接上R2S，并且用armbian-config命令打开配置界面
```
sudo armbian-config
```
1、选择“Network”的选项名，按下回车。

2、选择“IP”的选项名，按下回车。

3、选择你需要修改的连网口，wlan0指的是无线网络，eth0指的是有线网口，之后回车。

4、选择“Static”的选项名。并且按下回车。

（默认的DHCP指的是使用路由的地址分配服务器自动分配IP，Static指的是自己设置静止的IP，所以这里选择Static）

5、输入你需要的IP地址，按下回车即保存。

Address：IP地址（不知道设置什么IP的话，在这里用DHCP之前分配的IP固定即可）

Netmask：网络掩码（默认为24，不了解的同志使用默认的网络掩码）

Gateway：网关地址（不懂的同志在这里用默认网关地址即可）

DNS：DNS服务器IP（建议设置为网关地址或者223.5.5.5）

## 更换国内源
1、以清华源为例编辑 /etc/apt/sources.list 文件
```
nano /etc/apt/sources.list
```
注释官方源并添加清华源
```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-security main restricted universe multiverse
```
2、编辑 /etc/apt/sources.list.d/armbian.list 文件
```
nano /etc/apt/sources.list.d/armbian.list
```
将 http://apt.armbian.com 替换为以下链接
```
https://mirrors.tuna.tsinghua.edu.cn/armbian
```
3、更新软件列表
```
apt update
```
## 更改 cpu 调度模式为 schedutil
系统默认模式为 onedemand，且在 armbian-config 中更改模式不生效。 编辑文件 nano /etc/default/cpufrequtils 并修改为
```
ENABLE=true
MIN_SPEED=816000
MAX_SPEED=1296000
GOVERNOR=schedutil
```
重启 cpufrequtils 服务
```
systemctl daemon-reload
systemctl restart cpufrequtils
```
## 
