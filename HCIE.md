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

> Mac 地址飘逸是指交换机上一个 VLAN 内有两个端口学习到同一个 MAC 地址，后学习到的 MAC 地址表项覆盖原 MAC 地址表项的现象。

###	概述

当一个 MAC 地址在两个端口之间频繁发生迁移时，即会产生 MAC 地址漂移现象。出现这种现象一般都意味着网络中存在环路，或者存在网络攻击行为。



###	原理

防止 MAC 地址漂移：

- 配置端口 MAC 地址学习优先级。
- 配置不允许相同优先级接口 MAC 地址漂移。

MAC 地址漂移检测

- 基于 VLAN 的 MAC 地址漂移检测。
- 全局 MAC 地址漂移检测。



###	命令示例



```shell
# 配置接口学习 MAC 地址的优先级; 缺省情况下优先级为 0
[SW-G0/0/1] mac-learning priority <priority-id>

# 配置禁止 MAC 地址漂移时报文的处理动作为丢弃; 缺省处理动作是转发
[SW-G0/0/1] mac-learning priority flpping-defend action discard

# 禁止相同优先级的接口发生 MAC 地址漂移
[SW] undo mac-learning priority <priority-id> allow-flapping

# 配置 MAC 地址漂移检测的 VLAN 白名单
[SW] mac-address flapping detection exclude vlan {xx}

# 配置发生漂移后接口的处理动作
[SW-G0/0/1] mac-address flapping action { quit-vlan | error-down }


# 查看 MAC 地址漂移记录
[SW] display mac-address flapping record

```





##	MACsec

> 提供二层数据安全传输



###	概述

MACsec 定义了基于以太网的数据安全通信的方法，通过逐跳设备之间数据加密来保证数据传输安全性。对应标准为 802.1AE。

典型应用场景如在接入交换机和上联的汇聚或核心交换机之间部署。



###	原理

在两台设备上预配置相同的CAK，两台设备会通过 MKA 协议选举出一个 Key Server；Key Server 决定加密方案，Key Server 会根据 CAK 等参数使用某种加密算法生成 SAK 数据密钥并发放给对端，这样两台设备拥有相同的 SAK 数据密钥，实现对称性的加密解密。



##	流量抑制和风暴控制









##	DHCP Snooping 

> DHCP Snooping 是 DHCP 的一种安全特性，用于保护 DHCP 客户端从合法的 DHCP 服务器获取 IP 地址，并记录 DHCP 客户端 IP 和 MAC 地址等参数的对应信息，防止针对DHCP 的攻击。



###	

###	术语

- DHCP Snooping 信任功能

- DHCP Snooping 绑定表



### 原理

二层接入设备启用了 DHCP Snooping 功能后，

- 从 DHCP ACK 报文中提取关键信息：PC 的 MAC 地址，IP 地址，租期；
- 获取与 PC 连接的开启 DHCP Snooping 功能的接口信息（包括接口编号和接口所属的 VLAN）

通过这些信息生成 DHCP Snooping 绑定表。由于绑定表记录了对应关系，故通过对绑定表的匹配检查能够有效防范非法用户的攻击。

DHCP Snooping 绑定表根据 DHCP 租期进行老化，或根据用户释放 IP 地址 时发出的 DHCP Release 报文自动删除对应表项。



###	命令示例



```shell
# 全局开启 DHCP Snooping 功能
[SW] dhcp snooping enable [ipv4 | ipv6]
# VLAN  视图下开启 DHCP Snooping 功能
[SW] dhcp snooping enable

# VLAN 视图下配置接口为信任状态
[SW] dhcp snooping trusted interface <interface-type interfac-number>

# 接口视图下配置接口为信任状态
[SW-G0/0/1] dhcp snooping trusted

```















##	IP Source Guard











##	防火墙高级特性：双机热备，虚拟系统







##	总结



- 端口隔离实现同一 VLAN 内端口之间的隔离。端口隔离模式可以分为 l2 和 all。
- 交换机 MAC 地址表可以分为静态 MAC 地址表，动态 MAC 地址表和黑洞 MAC 地址表。
- 端口安全通过将端口学习到的动态MAC地址转换为安全 MAC 地址或 Sticky MAC 地址，安全 MAC 地址通常与安全保护动作结合使用。
- 开启 MAC 地址漂移检测有助于快速处理环路和被攻击的情况。
- MACsec 定义了基于以太网的安全数据通信的方法，通过逐跳设备之间数据加密，保证数据传输安全性。
- 流量抑制与风暴控制主要区别在于流量控制仅仅是对各种报文进行限速处理，超过阈值之后就丢弃；而风暴控制能够根据报文速率的大小采取不通的惩罚动作，包括关闭端口或者阻塞报文。
- DHCP Snooping 技术对于防御以太网中关于终端设备自动获取 IP 的网络攻击有很大的作用，通过配置 DHCP Snooping 信任端口和 DHCP Snooping 绑定表，可以很好的防范针对 DHCP 的 网络攻击。
- IPSG 通过查看交换机绑定表，阻止 IP 地址欺骗攻击，杜绝非法用户盗用合法 IP 进行攻击。
- 部署防火墙双机热备可提升网络可靠性，部署防火墙虚拟系统可实现对物理设备的逻辑划分，提升资源利用率。