####	配置文件导入

1. 配置交换机管理口IP地址：

   ```bash
   <H3C> system-view
   [H3C] interface Vlan-interface 1
   [H3C] ip address 192.168.1.1 255.255.255.0
   ```

2. 使用 tftp 将电脑中得配置文件下载到交换机中：

   ```bash
   <H3C> tftp 192.168.1.10 get xxxx.cfg
   ```

3. 更改配置文件命名：

   ```bash
   <H3C> dir
   <H3C> copy xxxx.cfg startup.cfg
   <H3C> startup saved-configuration startup.cfg
   <H3C> reboot
   ```

   

####	SSH批量导入配置

1. 配置用户名

   ```bash
   # ssh 连接用户的排序，从 0 ~ 5，可供 6 位用户同时登录
   [H3C] line vty 0 5
   [H3C-line-vty0-5] authentication-mode scheme
   # 老设备可能需要以下命令
   [H3C] user-interface vty 0 5
   [H3C] authentication-mode scheme
   
   
   
   # 创建本地用户，并进入本地用户视图
   [H3C] local-user gx class manage
   [H3C-luser-manage-gx] password simple guoxiang001
   # 为本地用户授权用户角色
   [H3C-luser-manage-gx] authorization-attribute user-role network-admin
   [H3C-luser-manage-gx] service-type ssh
   [H3C] ssh server enable
   ```

   

老旧设备的SSH版本过旧，客户端需要加参数来支持旧协议，例如：

```bash
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa -c aes128-cbc -o MACs=hmac-sha1 gx@192.168.10.1
```

