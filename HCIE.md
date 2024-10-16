#	二层网络安全技术



##	端口隔离

> VLAN 隔离广播域，而 VLAN 内隔离需要端口隔离



###	原理

通过在接口绑定隔离组，隔离组内的接口；隔离组内的接口不能相互通信。



###	术语

**端口隔离组 Port-isolate Group ：**在同一隔离组内的端口相互不能通信。

隔离类型：

- 单向隔离：同一隔离组且同一设备内，配置接口单向隔离，则该接口无法向同组其他接口转发数据，但反之可以。 
- 双向隔离：缺省情况是双向隔离。

隔离模式：

- L2 ：二层隔离三层通信，缺省情况下是 L2；这种模式下在 VLANIF 接口上使能 VLAN 内 Proxy ARP 功能，配置 arp-proxy inner-sub-vlan-proxy enable，可以实现同一VLAN内主机通信。
- ALL：二层三层都隔离。



###	命令示例

```shell
[s1] undo info-center enable	# 关闭系统日志

[s1-GigabitEthernet0/0/1] prot-isolate enable <1-64>	# 在端口上启用隔离组

[s1-GigabitEthernet0/0/1] am isolate G 0/0/2	# 指定单向隔离

[s1] port-isolate mode [all | l2]	# 设置端口模式
```



###		详细配置



## MAC地址表安全





###	术语：

**MAC地址表项：**

- 动态MAC地址表项：由接口通过报文的源MAC地址学习获得，表项可老化。系统复位，接口板热插拔或接口复位后，动态表项会丢失。
- 静态MAC地址表项：手工配置，不老化，保存的表项不会丢失。接口和MAC地址静态绑定后，其他接口收到源MAC是该MAC地址的报文将会被丢弃（高优先级）。
- 黑洞MAC地址表项：手工配置，不老化，源或目的MAC地址是该MAC的报文将会被丢弃。



### 原理

MAC地址表安全功能：

- 静态MAC地址表项：将一些固定的上行设备或信任用户的MAC地址配置为静态MAC表项，可以保证其安全通信。
- 黑洞MAC地址表项：防止黑客通过MAC地址攻击网络，交换机对来自黑洞或者去往黑洞的报文采取丢弃处理。
- 动态MAC地址老化时间：合理配置配置时间防止MAC地址爆炸式增长。
- 禁止MAC地址学习功能：可以限制非信任用户接入。
- 限制MAC地址学习数量：防止攻击者通过变化MAC地址进行攻击。



### 命令示例：



```shell
[S1] mac-address static xxxx-xxxx-xxxx G 0/0/3 vlan 1	# 静态MAC配置

# 动态MAC地址是通过流量触发的, 会将SMAC与接口进行对应
[S1] mac-address blackhole 5489-9812-1212 vlan 1

[S1] mac-address aging-time <0, 10-1000000>	# 设置MAC地址表项的老化时间

# 设置接口动态MAC地址的学习数量， 如果超出了配置的值，交换机就按照未知单播帧泛洪
[S1-G0/0/3] mac-limit maximum 2 

[S1-G0/0/3] mac-address learning disable	# 设置接口不学习动态MAC地址

# 对于接口没有保存的MAC地址，该数据执行丢弃
[S1-G0/0/3] mac-address learning disbale action discard

```





##	端口安全

> ​	企业要求接入层交换机上每个连接终端设备的接口均只允许一台PC接入网络（限制MAC地址接入数量）；要求只有MAC地址为可信任的终端发送的数据帧才允许被交换机转发到上层网络；还可能要求员工不能私自更换位置（变更交换机的接入端口）。这些都可以通过交换机的**端口安全（port security）特性**来解决



###	概述

部署端口安全可以：

- 限制接口 MAC 地址学习数量，并配置出现越时限的惩罚措施。
- 阻止非法用户通过本接口和交换机通信，从而增强设备的安全性。



###	原理

端口安全通过将接口学习到的MAC地址转换为安全MAC地址：

- 安全 Dynamic MAC：开启端口安全但未开启 Sticky MAC；缺省情况下不会被老化。
- 安全 Static MAC：开启端口安全后手动配置的静态 MAC。
- Sticky MAC：开器端口安全又开启 Sticky MAC；不会被老化，重启不丢失；由合法的动态 MAC 转换而来。

安全 MAC 地址通常和安全保护动作结合使用，常见的动作：

- Restrict：丢弃源 MAC 地址不存在的报文，上报告警。
- Protect：只丢弃源 MAC 地址不存在的报文，不上报告警。
- Shutdown：接口状态被置为 error-down，并上报告警。



###	术语

**安全 MAC 地址**

**Sticky MAC 地址**





###	应用场景

1. 控制接入用户数量，开启端口安全功能并指定安全 MAC 地址的限制数量。
2. 将大量固定用户配置为 Sticky MAC 地址，设备重启表项不丢失。
3. 将少量固定用户配置为 Security Static MAC 地址，设备重启表项不丢失。
4. 将频繁接入的用户配置为 Security Dynamic MAC 地址，易于清除绑定的表项。

 配置端口安全和动态MAC地址学习数量限制后，接口学习的MAC地址会被转换为 Security MAC 地址。接口学习达到上限后便不在学习新的MAC地址，仅允许这些MAC地址和交换机通信。如果收到源MAC地址不存在的报文，无论目的MAC地址是否存在，交换机即认为有非法用户攻击，就会根据配置的动作对接口做保护处理。这样可以阻止其他非信任用户通过本接口和交换机通信，提高交换机与网络的安全性



###	命令示例

```shell
[S1-G0/0/1] port-security enable	# 使能端口安全功能
[S1-G0/0/1]	port-security max-mac-num <max-number>	# 配置安全动态MAC学习限制, 缺省情况下限制为 1

# 手工配置安全静态 MAC 地址表项
[S1-G0/0/1]	port-security mac-address <mac-address> vlan <vlan-id>

# 配置端口安全保护动作, 缺省动作是 restrict
[S1-G0/0/1] port-security protect-action {protect | restrict | shutdown}
```



```shell
# 配置接口学习到的安全动态MAC地址的老化时间
[S1-G0/0/1] port-security aging-time <time> [type {absolute | inactivity}]

# 开启接口 Sticky MAC 功能
[S1-G0/0/1] port-security mac-address sticky

# 配置接口 Sticky MAC 学习限制数量, 缺省为 1
[S1-G0/0/1] port-security max-mac-num <max-number>

# 手动配置一条 sticky-mac 表项
[S1-G0/0/1] port-security mac-address sticky <mac-address> vlan <vlan-id>

```







##	MAC地址飘逸检测



##	风暴控制



##	端口限速







##	DHCP Snooping 和 IP Source Guard



##	防火墙高级特性：双机热备，虚拟系统