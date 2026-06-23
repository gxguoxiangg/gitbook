##	OpenWrt

OpenWrt（Open Wireless Router）是一个基于Linux的嵌入式操作系统开源项目，主要用于路由器。

OpenWrt 可通过命令行界面（ash shell）或 Web界面（LuCI）进行配置，有操作9000个可选软件包可通过软件包管理系统进行安装。

可以把一台 OpenWrt 路由器理解成一台微型 Linux 服务器：

```
硬件
 ↓
BootROM
 ↓
U-Boot
 ↓
Linux Kernel
 ↓
RootFS
 ↓
OpenWrt系统
 ↓
网络服务（dnsmasq、firewall、pppoe等）
```



####	固件 Firmware

固件：烧录到设备Flash芯片中的软件。



####	BootROM

CPU内部自带的一小段程序，它存放在芯片内部，无法修改。

作用：上电后，读取Flash，寻找、加载、执行 U-Boot。



####	U-Boot

全称 Universal Boot Loader

类似于PC里的BIOS+Windows Boot Manager

作用：

1. 初始化硬件
2. 加载Linux Kernel
3. 启动系统



####	Kernel

Linux 内核负责：

- CPU调度
- 内存管理
- 驱动管理



####	RootFS

全称 Root File System，根文件系统

相当于 Windows 的 C盘，里面有：

```
/bin
/etc
/lib
/usr
/sbin
```



####	Flash、RAM

Flash：路由器上的存储芯片。

RAM：运行内存。



####	Factory 分区

这里保持着：

- MAC地址
- WiFi校准数据
- 射频参数
- 序列号



####	Overlay



####	Recovery 模式



##	CMCC RAX3000M EMMC 算力版刷机示例

####	通过导入更改的配置文件获取root权限并开启telnet

EMMC版，配置文件可能是加密的，新批次加密密码是根据SN生成的，每台机器都不一样，且新版已删除了dropbear，只能开启telnet。

解密导出的配置文件：

```bash
# 替换成自己的SN
SN=081115000000000 
mypassword=$(openssl passwd -1 -salt aV6dW8bD "$SN")
mypassword=$(eval "echo $mypassword")
echo "$mypassword"
openssl aes-256-cbc -d -pbkdf2 -k "$mypassword"  -in cfg_export_config_file.conf -out 
cfg_export_config_file.conf.tar.gz 


# 解压解密后的配置文件
tar -xzvf cfg_export_config_file.conf.tar.gz
```

开启telnet：

```bash
telnetd -p 23 -l /bin/ash
echo 3 > /proc/sys/vm/drop_caches
```

修改shdows文件，删除root后面得密码：

```BASH
root:"irrversiable-password":xxx:xxx:
```

####	安装ssh

·telnet进入路由器后，更改包管理器配置：

```bash
cd /etc/opkg
# 备份配置文件
cp distfeeds.conf distfeeds.conf.bak
```

因为自带的wget不支持https，修改distfeeds.conf文件改为http模式下载wget-ssl，然后再修改回https：

```bash
opkg update
opkg install wget-ssl
```

安装ssh软件：

```bash
opkg update
opkg install openssh-server
opkg install openssh-client
```



参考链接：

1.https://github.com/lgs2007m/Actions-OpenWrt/blob/main/Tutorial/RAX3000M-eMMC_XR30-eMMC.md

2.https://blog.csdn.net/asdcls/article/details/147434218



####	刷入U-Boot

下载uboot：https://github.com/hanwckf/bl-mt798x/releases

将mt7981_cmcc_rax3000m-emmc-fip.bin上传至路由器。

写入：

```bash
dd if=mt7981_cmcc_rax3000m-emmc-fip.bin of=/dev/mmcblk0p3
sync
```



####	输入OpenWrt固件

端口路由器电源，按住Reset键不放，插入电源线，保持10秒后松开。修改主机网络配置：

- IP: 192.168.1.2
- MASK: 255.255.255.0
- Gateway 192.168.1.1
- DNS: 192.168.1.1

浏览器打开192.168.1.1，进入U-Boot界面，上传固件，例如https://fw.koolcenter.com/xiangfeidexiaohuo-easywrt/CMCC-RAX3000M-eMMC/中的EasyWrt-CMCC-RAX3000M-eMMC-5.4-20240324_ae79c0c365.bin。上传后写入。

<img src="https://qnam.smzdm.com/202403/27/6603eb9fe6a533067.png_e1080.jpg" alt="RAX3000Z增强版（RAX3000M EMMC版）刷机" style="zoom:33%;" />

####	进入OpenWrt管理页面

将主机IP设置修改回自动DHCP，访问192.168.2.1，账号为root，密码为空。进行其他配置。



####	恢复错误

如果系统的配置错误，内核和基本系统文件完好，可以用U-Boot的恢复模式救回来。

1.进入U-Boot恢复模式：

- 断开电源
- 网线插LAN1口，另一端连电脑。
- 电脑设固定IP：
  - IP: 192.168.1.254
  - MASK: 255.255.255.0
  - Gateway: 192.168.1.1
- 按住Reset键不松手，然后插电
- 继续按住约10-15秒，松开。

2.浏览器访问192.168.1.1。

3.重新上传刷入固件。

