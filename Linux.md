

####	安装Linux

以安装Ubuntu 22.04 Server版为例：

1. 下载镜像 https://mirrors.tuna.tsinghua.edu.cn/。
2. 使用 Rufus 将镜像刻录到U盘中。
3. 参照 https://blog.csdn.net/qq_46329367/article/details/156659239 进行初始安装。

网络配置：

1. 新建或编辑 `/etc/netplan/` 里的 yaml 配置文件。

2. 设置静态地址或动态地址：

   ```yaml
   # DHCP 获取地址
   network:
     version: 2
     renderer: networkd	# 后端设置为 system-networkd
     ethernets:
       enpxxx:				# 替换为实际的网络接口名称
         dhcp4: true		# 启用 IPv4 DHCP
   ```

   ```yaml
   network:
     version: 2
     renderer: networkd
     ethernets:
       enpxxx:								
         dhcp4: false				 
         addresses:
           - x.x.x.x/xx
         routes:
           - to: default
             via: x.x.x.x					# 设置默认网关
         nameservers:
           addresses: [x.x.x.x, y.y.y.y] 	# 设置DNS服务器
   ```

   如果服务器有多块网卡，需要连接不同的网络，可以分别配置它们：

   ```yaml
   network:
     version: 2
     renderer: networkd
     ethernets;
       eth0:
         dhcp4: false
         addresses:
           - x.x.x.x/xx
         routes:
           - to: default
             via: x.x.x.x
         nameservers:
           addresses: [x.x.x.x]
         eth1:
           dhcp4: false
           addresses:
             - x.x.x.x/xx
   ```

   

3. 保存文件后，应用配置：

   ```bash
   sudo netplan apply
   ```

   

4. 错误诊断，可以使用：

   ```bash
   sudo systemctl status systemd-networkd
   sudo journalctl -u systemd-networkd --no-pager -l -n 50
   ```



####	网络配置

**使用 nmcli 管理网络**

```bash
# 查看网络连接状态
nmcli device status
```

检查无线功能：

```bash
# 查看无线状态，如果显示 "yes" 表示已被阻塞
rfkill list

# 如果有阻塞，执行命令解锁
sudo rfkill unblock wifi

# 查看无线网卡是否开启
nmcli radio wifi

# 如果返回 "disabled"，用这个命令开启
nmcli radio wifi on
```

连接无线网络：

```bash
sudo nmcli dev wifi connect "YourSSID" password "YourPassword"
```





####	Python

在Linux环境下使用pip：

```bash
# 安装所有部件
sudo apt install python3-all
```

环境规范：

```bash
# 从Ubuntu 23.04开始, 保护系统的Python环境免受意外破坏
# 1.直接安装在系统环境下，版本可能不是最新的
sudo apt install python3-flask
# 2.创建虚拟环境
python3 -m venv .venv
source .venv/bin/activate
pip install flask
```

依赖文件：

```bash
# 自动生成当前环境的所有依赖
pip freeze > requirements.txt
# 仅扫描项目实际使用的依赖
pip install pipreqs
pipreqs . --force
```





####	Linux中使用U盘

1. 查看块设备

   使用 lsblk：

   ```bash
   lsblk
   NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
   sda      8:0    0 238.5G  0 disk
   ├─sda1   8:1    0   512M  0 part /boot/efi
   ├─sda2   8:2    0 237.1G  0 part /
   └─sda3   8:3    0   976M  0 part [SWAP]
   sdb      8:16   1   7.5G  0 disk          # ← U盘设备
   └─sdb1   8:17   1   7.5G  0 part          # ← U盘的分区
   ```

   分区名为：`/dev/sdb1`

2. 创建挂载点

   ```bash
   sudo mkdir /media/usb
   ```

3. 挂载U盘分区

   FAT32/exFAT格式U盘：

   ```bash
   sudo mount /dev/sdb1 /media/usb
   ```

   NTSF格式U盘：

   ```bash
   sudo apt install ntfs-3g
   sudo mount -t ntfs-3g /dev/sdb1 /media/usb
   ```

4. 安全卸载U盘：

   ```bash
   sudo umount /media/usb
   ```



####	更新时区

查看当前时间状态：

```bash
timedatectl
```

将时区设置为北京时间：

```bash
sudo timedatectl set-timezone Asia/Shanghai
```





#### MySQL相关

SQL备份：

```bash
mysql -u "用户名" -p "数据库" < 备份文件.sql
```





####	Nginx 相关

Nginx的配置文件和目录遵循特定的结构：

| 文件/目录                         | 作用                 | 说明                                                         |
| :-------------------------------- | :------------------- | :----------------------------------------------------------- |
| **`/etc/nginx/nginx.conf`**       | **主配置**           | 包含 Nginx 的全局配置，如工作进程数、运行用户、错误日志路径等。所有其他配置文件通常会被引入到此文件中 。 |
| **`/etc/nginx/sites-available/`** | **站点可用配置**     | 存放所有独立站点的配置文件（虚拟主机配置）。每个站点一个文件，例如 `example.com.conf`，但默认情况下它们并不会生效 。 |
| **`/etc/nginx/sites-enabled/`**   | **站点启用配置**     | 存放真正生效的站点配置文件。这里的文件通常是 `sites-available` 目录下对应文件的**符号链接（快捷方式）**。启用一个站点，就是在这里创建一个链接 。 |
| **`/var/log/nginx/access.log`**   | **访问日志**         | 记录所有对 Nginx 服务的访问请求 。                           |
| **`/var/log/nginx/error.log`**    | **错误日志**         | 记录 Nginx 运行过程中的错误信息，排查问题时通常从这里入手 。 |
| **`/etc/nginx/conf.d/`**          | **全局配置片段目录** | 存放一些被主配置文件引用的全局配置片段 。                    |







####	MRBS部署

使用 Nginx + MySQL + PHP。

1.安装PHP时，注意安装上必需扩展：

```bash
sudo apt install -y \
php-fpm \
php-mysql \
php-gd \
php-mbstring \
php-intl \
php-curl \
php-xml \
php-zip
```

2.创建MRBS数据库：

```sql
create database mrbs character set utf8mb4 collate utf8mb4_unicode_ci;
create user mrbs@localhost identified by '强密码';
grant all privileges on mrbs.* to mrbs@localhost;
flush privileges;
exit;
```

配置nginx虚拟主机：

```bash
sudo vim /etc/nginx/sites-available/mrbs
```

示例配置

```nginx
server {
    listen 80;
    server_name _;
    
    root /var/www/mrbs/web;
    index index.php index.html;

    access_log /var/log/nginx/mrbs.access.log;
    error_log  /var/log/nginx/mrbs.error.log;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires max;
        log_not_found off;
    }
}
```

启用站点：

```bash
sudo ln -s /etc/nginx/sites-available/mrbs /etc/nginx/sites-enable/
sudo nginx -t
sudo systemctl reload nginx
```

3.更改MRBS配置文件

```bash
sudo vim /var/www/mrbs/web/config.inc.php
```

```php
$dbsys = 'mysql';
$db_host = 'localhost';
$db_database = 'mrbs';
$db_login = 'mrbs';
$db_password = 'StrongPasswordHere';
```

4.安装语言环境

```bash
# 查看系统的环境
locale -a
# 安装更多语言环境
sudo locale-gen zh_CN.UTF-8
sudo update-locale
```



####	Tmux

**Tmux的dotfile默认在**

```bash
~/.tmux.conf	# 可以修改该文件进行自定义配置
```

tmux主要由会话，窗口，窗格（session，window，pane）组成



> 一个会话可以有多个窗口

创建新的窗口

```bash
ctrl-b c
```

切换窗口

```bash
ctrl-b <number>	# number : 窗口编号
```

切换上一个窗口

```bash
ctrl-b p
```

> 会话的处理

显示正在运行的会话列表

```bash
tmux ls
```

连接会话

```bash
tmux a -t 0
```

创建会话时指定名称（而不是像0这样的数字名称）

```bash
tmux new -s database
```

重命名现有对话

```bahs
tmux rename-session -t old_name new_name
```

> 其他常用命令

全屏显示窗格

```bash
ctrl-b z
```

调整窗格大小

```bash
ctrl-b ctrl-<arrow key>
```

重命名当前窗口

```bash
ctrl-b ,
```

将所有窗格等距排列

```bash
ctrl-b space
```



####	排障相关

查看内核日志：

```bash
# -T: 按照可读时间戳显示
dmesg -T
```







您好，深圳基地目前上网需要账号登录认证。基地后续工程验收、招商引资等会有来宾访问基地，存在上网需求。此类人员没有账号无法上网，请问是否有申请访客上网账号这类操作，或者有其他解决方案？烦请告知相关流程，谢谢。

