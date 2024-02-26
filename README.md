# R2S-Armbian
## Armbian 的安装与基础配置
以下内容基于 Armbian Jammy CLI 版本，默认在root权限下操作。

安装Armbian的第一步就像OpenWrt一样，从[官方网站](https://www.armbian.com/nanopi-r2s)下载对应的img（本文使用基于 Ubuntu 的 Jammy），然后用[工具](https://etcher.balena.io)烧录进SD卡。将R2S接入电源，指示灯应该闪烁，不会像OpenWrt一样表现为SYS红灯闪烁直至系统启动完毕红灯常亮，而是Heartbeat模式双闪。

初始设置需要将其WAN或LAN口接到另一台已经启用DHCP的路由器上，此时需在主路由管理员后台上查看Armbian内网地址，然后通过SSH连接进行初始化设置(初始用户名和密码分别为root和1234)。注意其此时不能作为一个开箱即用的路由器，也就是说不能直接接入电脑。

登入后有一个初始引导，跟着做就行了，没什么好说的。然后这就是一个标准的Linux系统了，基本可以直接遵循其他Linux的操作方法。但国内许多用户会到手换源，由于Armbian是基于其他发行版做的，所以换源不完全一样。其软件源有两个位置，均需要更改：

- /etc/apt/sources.list 与你安装的版本所基于的发行版有关，如Armbian Jammy基于Ubuntu Jammy，就换对应的源（如[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu-ports)，注意Ubuntu使用Ports源）；

- /etc/apt/sources.list.d/armbian.list 是 Armbian 专属软件源，也要换（仍然如[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/armbian)）。

## 使用系统内置的静态IP
使用SSH连接上R2S，并且用armbian-config命令打开配置界面
```
sudo armbian-config
```
1、选择"Network"的选项名，按下回车。

2、选择"IP"的选项名，按下回车。

3、选择你需要修改的连网口，之后回车。

4、选择"Static"的选项名。并且按下回车。

  (默认的DHCP指的是使用路由的地址分配服务器自动分配IP，Static指的是自己设置静止的IP，所以这里选择Static)

5、输入你需要的IP地址，按下回车即保存。

  Address：IP地址（不知道设置什么IP的话，在这里用DHCP之前分配的IP固定即可）

  Netmask：网络掩码（默认为24，不了解的同志使用默认的网络掩码）

  Gateway：网关地址（不懂的同志在这里用默认网关地址即可）

  DNS：DNS服务器IP（建议设置为网关地址或者223.5.5.5）

6、按下回车之后查看路由器后台，或者看设置里的IP位置即可。

## 更换国内源
1、以清华源为例编辑 /etc/apt/sources.list 文件
```
nano /etc/apt/sources.list
```
注释官方源并添加清华源
```
# 默认注释了源码镜像以提高apt update速度，如有需要可自行取消注释
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
## 更改CPU调度模式为schedutil
系统默认模式为 onedemand，且在 armbian-config 中更改模式不生效。 编辑文件并修改
```
nano /etc/default/cpufrequtils
```
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
## 优化TCP窗口,开启内核转发(Tun模式)
```
wget --no-check-certificate -O tools.sh https://raw.githubusercontent.com/Moorigin/Linux/main/tools.sh && chmod +x tools.sh && ./tools.sh
```
下载速度或无法下载，通过能访问外网的设备去[项目地址](https://github.com/Moorigin/Linux)下载deb文件,然后通过sftp上传，然后执行bash tools.sh
```
bash tools.sh
```
不需要修改TCP窗口可直接执行下面命令开启IP转发
```
echo 1 > /proc/sys/net/ipv4/ip_forward
```
## 安装Sing-Box
- Debian/Ubuntu
```
bash <(curl -fsSL https://sing-box.app/deb-install.sh)
```
下载速度或无法下载，通过能访问外网的设备去[项目地址](https://github.com/SagerNet/sing-box/releases)下载deb文件,然后通过sftp上传安装
```
sudo dpkg -i sing-box.deb
rm sing-box.deb
```
### 文件目录规划
```
主程序：  /usr/bin/sing-box
工作目录：/var/lib/sing-box
配置目录：/etc/sing-box
服务文件：/etc/systemd/system/sing-box.service
```
### Sing-Box命令
```
启用	sudo systemctl enable sing-box
禁用	sudo systemctl disable sing-box
启动	sudo systemctl start sing-box
停止	sudo systemctl stop sing-box
强行停止	sudo systemctl kill sing-box
重新启动	sudo systemctl restart sing-box
查看状态	sudo systemctl status sing-box
查看日志	sudo journalctl -u sing-box --output cat -e
实时日志	sudo journalctl -u sing-box --output cat -f
```
## 编辑Sing-Box配置文件
```
nano /etc/sing-box/config.json
```
## 启动Sing-Box
```
sudo systemctl enable sing-box
sudo systemctl start sing-box
```
## 卸载Sing-Box
```
# --purge会把配置也删除，保留配置请移除该参数
sudo systemctl stop sing-box.service
sudo systemctl disable sing-box.service
sudo dpkg --purge sing-box
```
