# R2S-Armbian
## armbian 的安装与基础配置
以下内容基于 Armbian Jammy CLI 版本，默认在 root 权限下操作。

将烧录了 armbian 系统的 tf 卡插入机器通电启动，不会像 openwrt 一样表现为 sys 红灯闪烁直至系统启动完毕红灯常亮，而是 heartbeat 模式双闪。此时需在主路由上查看 armbian 内网地址后通过 ssh 进入进行初始化，并将网关设为主路由地址。

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
