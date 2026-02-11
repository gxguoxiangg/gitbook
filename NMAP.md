###	Network Mapper，简称Nmap

> 用户网络发现和安全审计的工具



####	扫描网段内的主机名

```bash
nmap -sU --script nbstat.nse -p 137 192.168.1.0/24 -oN nbstat_results.txt
```

功能包括：

1. 获取NetBIOS主机名
2. 获取MAC地址
3. 获取Windows工作组/域名
4. 识别Windows操作系统版本
5. 查看NetBIOS服务信息

输出示例：

```bash
$ nmap -sU --script nbstat.nse -p 137 192.168.1.100
PORT    STATE SERVICE
137/udp open  netbios-ns
MAC Address: 00:0C:29:XX:XX:XX (VMware)
Host script results:
| nbstat:
|   NetBIOS name: WIN10-PC, NetBIOS user: <unknown>, NetBIOS MAC: 00:0c:29:xx:xx:xx
|   Names:
|     WIN10-PC<00>           Flags: <unique><active>
|     WORKGROUP<00>          Flags: <group><active>
|     WIN10-PC<20>           Flags: <unique><active>
|     WORKGROUP<1e>          Flags: <group><active>
|   Statistics:
|     Received 5 packets, got 5 answers
|_    Request timed out 0 times
```



####	主机发现

只进行主机发现，不进行端口扫描

```bash
# 扫描网段
nmap -sn 192.168.1.0/24

# 扫描多个目标
nmap -sn 192.168.1.1,2,3,10-20
```

