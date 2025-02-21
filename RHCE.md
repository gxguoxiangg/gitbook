##	1.配置网络设置

主机名修改：

```bash
hostnamectl set-hostname xxx.xxx
```

修改 DNS：

```bash
chattr -i /etc/resolv.conf
```

查询网络配置文件：

```bash
nmcli connection show
```

修改网络配置文件：

```bash
nmcli connection mod 'Wired connection 1' \
ipv4.method manual \
ipv4.addresses 172.25.250.100/24 \
ipv4.gateway 172.125.250.254 \
ipv4.dns 172.25.250.254 \
autoconnect yes
```

启动配置好的网卡：

```bash
nmcli connection up 'Wired connection 1'
```

检查配置：

```bash
ip a
hostname
```



**PS:**

nmcli

添加网卡并配置静态IP地址：

```bash
nmcli con add con-name <连接名> ifname <网卡名> type <连接类型> \
ipv4.method manual \
ipv4.addresses <ipv4地址> ipv4.gateway <ipv4网关> ipv4.dns <ipv4 dns服务器> \
autoconnect yes
```

激活/关闭网卡：

```bash
nmcli con {up/down} <连接名>
```

查看网卡信息：

```bash
nmcli con show
```

修改网卡的地址：

```bash
nmcli con modify <连接名> ipv4.method manual ipv4.addresses <新ip地址> ipv4.gateway <新网关> ipv4.dns <dns服务器>
```

添加网卡配置为动态获取ip地址

```bash
nmcli con add con-name <连接名> ifname <网卡名> type <连接类型> ipv4.method auto
```

nmcli 删除一个网卡配置信息

```bash
nmcli con del <连接名>
```

nmcli 为一张网卡配置多个ip地址（从地址）
```bash
nmcli con modify <连接名> +ipv4.address <ip地址> ipv4.gateway <网关> ipv4.dns <dns地址>
```





##	2.配置系统默认存储库(配置yum本地源)

更改配置文件:

```bash
vim /etc/yum.repos.d/xxxx.repo
```

vim:

```bash
[xxx]	# repo id
name=Base
baseurl=xxxx
enabled=1
gpgcheck=no
[xxx]	# repo id
name=App
baseurl=xxxx
enabled=1
gpgcheck=no
```

检查:

```bash
yum repoinfo
yum -y install vsftpd
```

较新的检查：

```bash
# 清空原有的仓库缓存
dnf clean all
# 生成新的缓存
dnf makecache
# 验证仓库
dnf repolist all
```



##	3.调试 SELinux

> SELinux，最早由美国国安局开发，并集成到 Linux 内核中。主要作用是提高系统安全性，限制进程对系统资源的访问，即使攻击者获取了某个进程的权限，也能最大程度减少损害范围。

SELinux 关键概念：

1. 三种模式

   - Enforcing（强制模式）：严格执行 SELinux 安全策略，阻止未经授权的访问。
   - Permissive（宽容模式）：不阻止访问，但会记录警告信息。
   - Disabled（禁用模式）：完全关闭 SELinux。

2. 安全上下文

   SELinux 使用标签来标识文件、进程和端口等资源，格式一般为：

   ```bash
   user:role:type:level
   ```

   - `user`：SELinux 用户
   - `role`：SELinux 角色（主要用于 RBAC）
   - `type`：类型（最重要的部分，用于定义访问规则）
   - `level`：可选的 MLS/MCS 安全级别

3. 策略类型

   - Targeted（目标策略）：默认策略，仅对特定进程应用 SELinux 规则。
   - MLS（多级安全策略）：用于高安全环境，支持更复杂的访问控制。

   



增加 SELinux 放行策略：

```bash
# 添加自定义的端口标签规则
semanage port -a -t http_port_t -p tcp 82
# 更改文件 SELinux 类型
chcon -t httpd_sys_content_t /var/www/html/*	# 可选操作，可能默认就已经是该类型
# 查看系统默认的 SELinux 文件上下文
semanage fcontext -l | grep "xxx"
```

检查标签是否更改：

```bash
# 检查文件的标签 type 是否修改为 httpd_sys_content_t
ls -l --context /var/www/html/
```

启动服务并设置为开机自启：

```bash
systemctl restart httpd.service
systemctl enable --now httpd.service
```

验证效果：

```bash
curl xxx/xxx
systemctl is-enabled httpd
```



##	4.创建用户账户



创建 sysmgrs 组：

```bash
groupadd sysmgrs
```

创建用户 natasha，作为次要组从属于 sysmgrs：

```bash
useradd -G sysmgrs 'natasha'
useradd -G sysmgrs 'harry'
```

创建用户 sarah，无权访问系统上的交互式 shell 且不是 sysmgrs 的成员：

```bash
useradd -s /bin/false 'sarah'
```

natasha、harry 和 sarah 的密码应当都是 flectrag

```bash
echo flectrag | passwd --stdin 'natasha'
```

检查：

```bash
id natasha
```



**PS**：

1. 通过 useradd 添加的用户默认是没有密码的，且不允许登录；会在 passwd 文件中创建添加的账户信息，且在 shadow 中添加用户，但不会设置密码，记录类似：

   ```bash
   # !! 表示账户已被锁定
   testuser:!!:19755:0:99999:7:::
   ```

   



##	5.配置周期性计划任务



**以用户 natasha 的身份每周三下午15:30运行logger"xxx"**



配置 crontab

```bash
crontab -u natasha -e
30 15 * * 3 logger "This is a rhcsa exam"
```

检查配置

```bash
crontab -u natasha -l
```



**PS**：

每隔多久运行，即频率；可以使用 `/`，如每隔 2 分钟运行一次：

```bash
*/2 * * * * commands
```





##	6.创建共享目录

创建具有以下特点的共用目录：

-  /home/tools 的所有组是admins
- 此目录能被admins组的成员读取、写入和访问
- 除root外其他用户没有这些权限在此目录下创建的文件，其组的所有权自动设置为admins组



使用 `chgrp` 更改所属组：

```bash
# 设置拥有组
chgrp admins /home/tools
# 设置组的权限
chmod g+rwx, o=--- /home/tools
# 设置 SGID
chmod g+s /home/tools

```





**PS**：

特殊权限有：

1. SUID（Set User ID)

   让用户执行某个可执行文件时，以**文件所有者**的身份运行，而不是当前用户的身份。主要用于需要临时提升权限的程序，如 passwd。

   ```bash
   # passwd 的权限显示如下
   -rwsr-xr-x 1 root root 12345 Feb 20 12:00 /usr/bin/passwd
   # 设置 SUID
   chmod u+s filename
   # 或
   chmod 4755 filename
   # 移除 SUID
   chmod u-s filename
   ```

2. SGID（Set Group ID）

   - 对可执行文件：

     让执行该文件的用户以**文件所属组**的权限运行，而不是用户自己的组

   - 对目录：

     目录下新创建的文件会继承该目录的**所属组**，而不是创建者的默认组。

   ```bash
   # 权限显示如下
   -rwxr-sr-x 1 root staff 12345 Feb 20 12:00 somefile
   
   # 设置 SGID
   chmod g+s 'filename 或 dirname'
   chmod 2755 'xxx'
   ```

3. Sticky Bit（粘滞位）

   仅使用于目录，防止用户删除其他用户的文件，即使它们有写权限。典型用例：`/tmp`目录，多个用户共用，但不能删除别人的文件。

   ```bash
   # 权限显示如下
   drwxrwxrwt 10 root root 4096 Feb 20 12:00 /tmp
   
   # 设置 Sticky Bit
   chmod +t 'dirname'
   chmod 1777 'dirname'
   ```

   

| 特殊权限       | 作用                                                         | 适用对象         | 典型用途            | 设置方式                     |
| -------------- | ------------------------------------------------------------ | ---------------- | ------------------- | ---------------------------- |
| **SUID**       | 让用户以**文件所有者身份**执行                               | 可执行文件       | `passwd`, `sudo`    | `chmod u+s filename`         |
| **SGID**       | 让用户以**文件所属组身份**执行（对文件）；**目录中新文件继承组**（对目录） | 可执行文件、目录 | `crontab`, 共享目录 | `chmod g+s filename/dirname` |
| **Sticky Bit** | 允许**仅文件所有者删除**文件                                 | 目录             | `/tmp` 目录         | `chmod +t dirname`           |



##	7.配置网络时间同步

配置系统成为指定的NTP客户端



安装 chrony：
```bash
yum -y install chrony
```

更改配置：

```bash
echo "server xxx.xxx.xxx iburst" >> /etc/chrony.conf
```

重启时钟同步服务：

```bash
systemctl restart chronyd
systemctl enable chronyd
```



##	8.配置文件系统自动挂载

按照以下要求，在serverA上配置autofs自动挂载： 

- serverB通过NFS共享目录/rhome到你的系统，此文件系统中包含为用户remoteuser0预配置的家目录 
- 预设用户remoteuser0的家目录应自动挂载到本地的/rhome/remoteuser0目录
- 预设用户remoteuser0的家目录是serverb.lab.example.com:/rhome/remoteuser0 
- 挂载后的家目录必须可读写



在serverB上创建共享目录，并配置权限：

```bash
# -p: 按需新建父目录, 如果已存在则跳过
mkdir /rhome/remoteuser0 -p
chmod 777 /rhome/remoteuser0
```

配置NFS的共享：

```bash
echo '/rhome/remoteuser0 *(rw,sync)' > /etc/exports
systemctl restart nfs-server.service
```

配置防火墙策略：



在 serverA 创建用户，并修改用户的家目录：

```bash
useradd remoteuser0
vim /etc/passwd
remoteuser0:x:1002:1002::/rhome/remoteuser0:/bin/bash
```



正式考试：

按照 *autofs*：

```bash
yum install -y autofs
```

查看共享:

```bash
showmount -e serverb.lab.example.com
```

编写配置文件：

```bash
echo '/rhome /etc/nfs.auto' >> /etc/auto.master

# 编写 nfs.auto
vim /etc/nfs.auto
remoteuser0 -rw serverb.lab.example.com:/rhome/remoteuser0
```

启动服务并设置自启：

```bash
systemctl enable --noe autofs.service
```

验证效果：

```bash
# 切换用户
su - remoteuser0
```



**PS**:

NFS（网络文件系统）允许 Linux 及其通过网络访问远程服务器上的共享目录，就像本地目录一样。NFS 服务器共享的目录可以被多个客户端挂载，实现跨主机的文件访问。

*autofs* 允许按需自动挂载和闲置自动卸载远程文件系统，适用于 NFS 共享目录。

```bash
# 安装 autofs
yum install -y autofs
# 编辑 /etc/auto.master
/game	/etc/auto.nfs	# 这说明 /game 的自动挂载规则是由 /etc/auto.nfs 负责
# 创建 /etc/auto.nfs
'相对挂载点的目录名称' -rw, sync xxx.xxx.xx:/nfs/xxx
# 启动 autofs
systemctl enable --now autofs
# 检查
df -Th | grep nfs
```



##	9.配置用户账户

配置用户账户 john，用户的 ID 为 2023 ，此用户的密码应当为 redhat

```bash
useradd john -u 2023 -p redhat
# 或
useradd john -u 2023
echo redhat | passwd --stdin john
```



##	10.查找文件

查找属于john用户所属的文件，并拷贝到 /root/findfiles 目录

```bash
# 创建目录
mkdir -p /root/findfiles
# 查找
find / -user john -exec cp {} /root/findfiles \; 2> /dev/null
```

**PS**：

*cp* 的 `-a` 选项：

1. 用来复制文件和目录，并保留文件的属性信息。即我想要复制这个文件或目录，并且我想保留所有与之相关的属性，就像它是被归档或备份的那样。
2. 等同于：`-dR --preserve=all`



##	11.查找字符串

找出文件 /etc/man_db.conf 中包含字符串 sbin 的所有行，将其按原始顺序导入到文件/root/out.txt 中，文件/root/out.txt中不得包含空行

```bash
mkdir -p /root
grep 'sbin' /etc/man_db.conf > /root/out.txt 
```



##	12.创建归档

创建一个名为 /root/backup-YYYY-MM-DD.tar.bz2 格式的 tar 包，用来压缩/usr/local目录进行备份。

```bash
tar -cjf /root/backup-$(date +%F).tar.bz2 /usr/local 
```

**PS**：

tar 的压缩选项：

- gzip：-z
- bzip2：-j
- xz：-J



##	13.创建一个容器镜像



```bash
dnf -y install container-tools
ssh user@node1
```

登录镜像仓库：

```bash
podman login -u admin -p redhat321 registry.lab.example.com
```

下载容器文件，并创建镜像：

```
wget xxx/xxx
podman build -t pdf .
```

验证：

```bash
podman images
```



##	14.配置容器开机自启

在 servera 上创建一个 rootless 容器，并配置为systemd服务自动启动，要求如下:

-  容器名为 txt2pdf，容器使用在其他项目中创建的 modify_file 镜像
- 该服务面向 sashat 以systemd 服务运行
- 服务名称为container-txt2pdf 
- 系统重启后，容器无需干预自动运行
- 将本地目录/opt/txt 附加到容器的 /opt/txt 目录
- 将本地目录/opt/pdf附加到容器的 /opt/pdf 目录
- 容器在运行情况下，/opt/txt下的任何纯文本文件将自动转换为pdf文件，并使用相同文件名并加上后缀pdf存放到 /opt/pdf下



主机上创建映射的目录：

```bash
mkdir /opt/{txt,pdf}
chown sashat:sashat /opt/{txt,pdf}
```

运行容器：

```bash
su - sashat
podman run -d --name txt2pdf -v /opt/txt/:/opt/txt:Z -v /opt/pdf:/opt/pdf:Z localhost/modify_file:latest
```

创建服务单元：

```bash
mkdir -p ~/.config/systemd/user
cd ~/.config/systemd/user

podman generate systemd txt2pdf --files --name --new
```

配置服务开机自启：

```bash
systemctl --user daemon-reload
systemctl --user enable container-txt2pdf.service
```

验证：

```bash
podman rm -f txt2pdf
systemctl --user restart container-txt2pdf.service
podman ps
```





##	15.添加sudo免密操作

允许 sysmgrs 组成员 sudo 时不需要密码

配置sudo文件：

```bash
vim /etc/sudoers.d/sysmgrs
%sysmgrs ALL=(root) NOPASSWD:ALL
```



**PS**：

sudoers 中的配置规则：

```bash
username ALL=(ALL) NOPASSWD:ALL
%groupname ALL=(ALL) NOPASSWD:ALL
```

- `username`：需要免密的用户名
- `ALL`：可以在所有主机上执行
- `(ALL)`：可以以任何用户身份执行命令
- `NOPASSWD:ALL`：允许所有命令免密执行





##	16.设置默认密码策略

为新创建的用户设置密码策略，要求创建用户时，密码默认20 天后过期。

编辑 /ect/login.defs 文件：

```bash
# 修改该项
PASS_MAX_DAYS 20
# 或
sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS 180/' /etc/login.defs
```

验证：

```bash
useradd user1
chage -l user1
```



**PS:**

`/etc/login.defs` ：

- 用户 UID/GID 号的默认范围。
- 密码过期策略
- 创建用户时的默认设置，umask
- 邮件目录和环境变量
- 密码加密算法



##	17.创建用于系统监控的脚本

- 创建一个名为 exam_sysinfo 的脚本
- 该脚本放置在 /usr/local/bin 下
- 该脚本用于获取当前系统进程的信息，要求按照顺序输出
  - 进程的所有者
  - 进程的 PID
  - 进程消耗的虚拟内存和实际内存
  - CPU的百分比并其中以cpu的百分比 进行排序，消耗CPU最多的进程在最后显示



```bash
vim exam_sysinfo

ps aux | sed '1d' | sort -k 3n | awk '{printf"%s %s %s %s %s ", $1,$2,$5,$6,$3}'
```



```bash
#!/bin/bash
ps axo user,pid,vsz,rss,%cpu --sort pcpu
```



##	18.运行一个容器

1. 在 http://content.example.com 中提供一个 rsyslog.tar 的镜像文件
2. 将镜像存放在 https://registry.lab.example.com/library 仓库中 
3. 使用 https://registry.lab.example.com/library/rsyslog 镜像运行名为logserver的容器 
4. 将生成的日志存放在 /home/sashat/syslog/目录下，并使用logger命令发送 "This is a rhcsa exam" 日志信息并在/home/sashat/syslog/message的文件中找到



下载镜像文件：

```bash
wget http://content.example.com/rsyslog.tar
```

导入镜像：

```bash
podman load < rsyslog.tar
```

标记 tag：

```bash
podman image tag localhost/rsyslog:latest registry.lab.example.com/library/rsyslog:latest
```

推送到仓库：

```bash
podman login registry.lab.example.com	# 登录仓库
podman push registry.lab.example.com/library/rsyslog:latest
```

运行容器：

```bash
mkdir ~/syslog
podman run -d --privileged -v ~/syslog:/var/log:Z --name logserver registry.lab.example.com/library/rsyslog:latest
```

进入容器执行命令：

```bash
podman exec -it logserver /bin/bash
[root@db2d8a1dcc70 ~] logger "this is a rhcsa exam"
```

验证效果：

```bash
grep rhcsa ~/syslog/messages
```





##	19.重置 root 密码

将 serverb.lab.example.com 主机的root密码设置成 Young

1. 重启按 F8 进入 grub 菜单

2. 选择第一个，按 E 进入编辑

3. 更改内核选项，在内核行末，加上

   ```bash
   init=/bin/bash
   ```

4. 按下 ctrl-x 重新进入系统

5. 更改权限

   ```bash
   # 当前只有只读权限 ro
   grep "/" /proc/mounts
   # 更改挂载的权限
   mount -o remount,rw /
   ```

6. 备份 /etc/shadow 的安全上下文

   ```bash
   ls -lZ /etc/shadow
   ```

7. 更改密码

   ```bash
   passwd
   ```

8.  还原上下文

   ```bash
   chcon system_u:object_r:shadow_t:s0 /etc/shadow
   ```

9. 更改为启动进程

   ```bash
   exec /sbin/init
   ```

   



##	20.配置yum源

请配置serverb的YUM仓库地址如下： 

- http://content.example.com/rhel9.0/x86_64/dvd/BaseOS
- http://content.example.com/rhel9.0/x86_64/dvd/AppStream

使用配置您的系统，以将这些位置用作默认存储库



创建配置文件 *.repo*

```bash
vim /etc/yum.repos.d/local.repo
```

编写配置

```bash
[base]
baseurl=http://content.example.com/rhel9.0/x86_64/dvd/BaseOS
name=BaseOS
gpgcheck=0
enabled=1
[AppStream]
baseurl=http://content.example.com/rhel9.0/x86_64/dvd/AppStream
name=AppStream
gpgcheck=0
enabled=1
```

更新仓库缓存

```bash
dnf clean all
dnf makecache
dnf repolist all
```





##	21.调整逻辑卷大小

- 将名为 rhel 的逻辑卷的大小调整到 512M，确保文件系统的内容保持不变。
- 调整后的逻辑卷的大小范围在 498M 到 511M 的范围内都是可以接受的。



扩容逻辑卷和文件系统

```bash
lvextend -r -L 512M /dev/exam/rhel
```

如果不使用 `-r` 选项，或者文件系统不支持在线调整大小，需要手动调整，可以使用 `resize2fs`

```bash
resize2fs /dev/xxx/xxx
```

验证效果

```bash
df -Th /mnt/rhel
```



##	22.添加交互分区

- 向您的系统添加⼀个额外的交换分区 512MiB 。交换分区应在系统启动时自动挂载 。不要删除或 以任何方式改动系统上的任何现有交换分区。

新建分区

```bash
fdisk /dev/vdb
```

格式化为 swap

```bash
mkswap /dev/vdb2
```

配置自动挂载

```bash
echo "/dev/vdb2 sawp swap defaults 0 0" >> /etc/fstab
```

激活 swap

```bash
swapon -a
```

验证效果

```bash
free -m
```



##	23.创建逻辑卷

根据如下要求，创建新的逻辑卷：

-  逻辑卷取名为 qa，属于 qagroup 卷组，大小为 60 个扩展块 
- qagroup 卷组中逻辑卷的扩展块大小应当为 16 MiB 
- 使⽤ vfat 文件系统格式化新逻辑卷。该逻辑卷应在系统启动时自动挂载到 /mnt/qa 下



创建卷组并设置 PE 大小为 16M

```bash
vgcreate -s 16M qagroup /dev/vdb3
```

创建逻辑卷

```bash
lvcreate -l 60 -n qa qagroup
```

格式化文件系统

```bash
mkfs.vfat /dev/qagroup/qa
```

配置自动挂载

```bash
echo '/dev/qagroup/qa' /mnt/qa vfat defaults 0 0
mount -a
```



**PS:**

LVM（logical volume manager）管理步骤如下：

1. 创建PV（物理卷）
   - fdisk建立新分区
   - pvcreate在新分区上创建物理卷
2. 创建VG（创建卷组）
   - vgcreate创建卷组
3. 创建LV（逻辑卷）
   - lvreate
     - `-l`：指定 VG 中 PE 的个数
4. 格式化LV
5. 挂载
