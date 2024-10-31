---
title: HCIE
date: 2024-10-14 12:00:00
tags:
    - 笔记

---



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



###	术语

- DHCP Snooping 信任功能

- DHCP Snooping 绑定表



### 原理

二层接入设备启用了 DHCP Snooping 功能后，

- 从 DHCP ACK 报文中提取关键信息：PC 的 MAC 地址，IP 地址，租期；
- 获取与 PC 连接的开启 DHCP Snooping 功能的接口信息（包括接口编号和接口所属的 VLAN）

通过这些信息生成 DHCP Snooping 绑定表。由于绑定表记录了对应关系，故通过对绑定表的匹配检查能够有效防范非法用户的攻击。

DHCP Snooping 绑定表根据 DHCP 租期进行老化，或根据用户释放 IP 地址 时发出的 DHCP Release 报文自动删除对应表项。

常见的攻击手段有：

- DHCP 饿死攻击
- 改变 CHADD 值的 Dos 攻击



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



#	IP 路由基础



###	简介

路由就是数据通信网络中最基本的要素，路由信息就是指导报文发送的路径信息，路由的过程就是报文转发的过程。

根据路由目的地址的不通，路由可划分为：

- 网段路由：目的地为网段，子网掩码长度小于 32 位。

- 主机路由：目的地位主机，子网掩码长度等于 32 位。

根据目的地与该路由器是否直接相连，路由又可划分为：

- 直连路由
- 间接路由

根据目的地址类型的不通，路由还可以分为：

- 单播路由
- 组播路由



####	路由表和 FIB 表

路由器转发数据包的关键是路由表和 FIB 表，每个路由器都至少保存一张路由表和一张 FIB （Forwarding Information Base）表。路由器通过路由表选择路由，通过 FIB 表指导报文进行转发。

路由表

每台路由器都保存着一张本地核心路由表（即设备的 IP 路由表），同时各个路由协议也维护着自己的路由表。



####	路由迭代

路由必须有直连的下一跳才能够指导转发，但是路由生成时下一跳可能不是直连的，因此需要计算出一个直连的下一跳和对应的出接口，这个过程就是路由迭代。



####	路由协议的优先级

路由器分别定义了外部优先级和内部优先级，外部优先级是指用户可以手工为各路由协议配置的优先级。

**缺省外部优先级：**

| 路由协议的类型 | 外部优先级 |
| -------------- | ---------- |
| Direct         | 0          |
| OSPF           | 10         |
| IS-IS          | 15         |
| Static         | 60         |
| RIP            | 100        |
| OSPF ASE       | 150        |
| OSPF NSSA      | 150        |
| IBGP           | 255        |
| EBGP           | 255        |

除了直连路由外，各种路由协议的优先级都可由用户手工进行配置。

**路由协议内部优先级：**

| 路由协议的类型 | 路由协议的内部优先级 |
| -------------- | -------------------- |
| Direct         | 0                    |
| OSPF           | 10                   |
| IS-IS Level-1  | 15                   |
| IS-IS Level-2  | 18                   |
| Static         | 60                   |
| RIP            | 100                  |
| OSPF ASE       | 150                  |
| OSPF NSSA      | 150                  |
| IBGP / EBGP    | 200 / 20             |

路由选择时先比较路由的外部优先级，当不同的路由协议配置了相同的优先级后，系统会通过内部优先级决定哪个路由协议发现的路由将成为最佳路由。

除此以外，策略路由提供了一种依据用户指定的策略进行路由选择的机制，它能够在路由表已经产生的情况下，不按照路由表而是按照用户进行转发，其优先级高于直连，静态和动态路由协议产生的路由。



####	负载分担和路由备份



####	IP FRR

快速重路由 FRR （Fast Reroute）是指当物理层或链路层检测到故障时，将故障消息上报至上层路由系统，并立即启用一条备份链路转发报文。IP FRR 是一种快速实现路由备份的方式。

IP FRR 针对 IP  网络路由而设计，分为公网 IP FRR 和私网 IP FRR：

- 公网 IP FRR：用于保护公网路由器。
- 私网 IP FRR：用于保护 CE（Customer Edge）。



####	路由的收敛

路由收敛是指网络拓扑变化引起的通过重新计算路由而发现替代路由的行为。按优先级收敛是指系统为路由设置不同的收敛等级，分为 critical，high，medium，low 四种。

**缺省的公网路由收敛优先级：**

| 路由协议或路由种类            | 收敛优先级 |
| ----------------------------- | ---------- |
| Direct                        | high       |
| Static                        | medium     |
| OSPF 和 ISIS 的 32 位主机路由 | medium     |
| OSPF / ISIS / BGP             | low        |
|                               |            |



####	缺省路由

缺省路由（默认路由）是一种特殊路由，如果报文的目的地址不能与路由表的任何目的地址相匹配，那么该报文将选取缺省路由进行转发。如果没有缺省路由且报文的目的地址不在路由表中，那么该报文将被丢弃，并向源端返回一个 ICMP 报文，报告该目的地址或网络不可达。

通常情况下，管理员可以通过手工方式配置缺省静态路由，也可以使动态路由协议生成缺省路由，如 OSPF 和ISIS。





####	路由的 Router ID

Router ID 是设备通过指定的动态路由协议进行路由交互过程中，唯一标识自身的 32位 整数。Router ID 的格式和 IP 地址的格式是一样的。Router ID 包括设备的全局 Router ID 和动态路由协议的 Router ID。

Router ID 有手动配置和设备自动选取两种选取方式。在实际网络部署中，考虑到协议的稳定性，建议手工配置 Loopback 接口的 IP 地址作为 Router ID。

- 如果动态路由协议配置了 Router ID，则以该协议自身的配置为准
- 如果动态路由协议没有配置，则以全局 Router ID 为准。
- 如果以上均未配置，设备会从当前接口的 IP 地址中自动选取一个作为全局 Router ID：
  - 如果存在配置了 IP 地址的 Loopback 接口，则选择 Loopback 接口地址中的最大的作为全局 Router ID。
  - 如果设备上不存在 Loopback 接口或者存在 Loopback 接口但没有配置 IP 地址。则从其他接口的 IP 地址中选择最大的作为全局 Router ID（不考虑接口的 UP/DOWN状态）。

当且仅当被选为 Router ID 的接口 IP 地址被删除或者修改，设备才会重新选取 Router ID。和 OSPF DR 选举类似，也是终身制的。



#	静态路由配置



在创建静态路由时，可以同时指定出接口和下一跳。对于不同的出接口类型，也可以只指定出接口或下一跳：

- 点到点：指定出接口
- NBMA：指定下一跳
- 广播：指定下一跳

配置示例：

```bash
[AR] ip route-static xx-xx-xx-xx mask xx preference xx tag xx
```

在配置静态路由时，如果没有指定优先级，就会使用缺省优先级。

```bash
# 配置静态路由的缺省优先级
[AR] ip route-static default-preference xx
```

####	静态路由按迭代深度优先选择

```bash
# 配置静态路由按迭代深度进行优先选择，优先选择深度小的
[AR] ip route-static selection-rule relay-depth
# 缺省情况下，静态路由不按迭代深度进行优选
```



####	静态路由永久发布

静态路由永久发布就是通过 Ping 静态路由目的地址的方式来检测链路的有效性。配置静态路由永久发布后，静态路由会一直生效，不受路由出接口状态的影响。这种情况下，系统只能通过指定路径转发 Ping 报文，不会从其他路径绕行，从而真实反映指定路径的链路状态。

缺省情况下，没有配置静态路由永久发布。

```bash
[AR] ip route-static xx-xx-xx-xx .... permanent
```





#	IGP 高级特性



##	OSPF 基础



###	概述

开放式最短路径优先 OSPF （Open Shortest Path First）是一个基于链路状态的内部网关协议（IGP）。

- OSPF 把自治系统 AS 划分成逻辑意义上的一个或多个区域。
- OSPF 通过链路状态通告 LSA （Link State Advertisement）的形式发布路由。
- OSPF 依靠在 OSPF 区域内各设备间交互 OSPF 报文来达到路由信息的同一。
- OSPF 报文封装在 IP 报文内，可以采用单播或组播的形式发送。



###	原理基础



####	OSPF 运行机制：

1. 通过交互 Hello 报文形成邻居关系。
2. 通过泛洪 LSA 通告链路状态信息。
3. 通过组建 LSDB 形成带权有向图。
4. 通过 SPF 算法计算并形成路由。
5. 维护和更新路由表。



####	OSPF 报文类型：

1. **Hello 报文：**
   - 邻居发现，建立邻居关系。
   - 指定 DR 和 BDR。
   - 保活。
2. **DD 报文：**
   - 初始化邻接关系时，DD 报文 （Database Description packet）用来协商主从关系，此时报文中只包含 LSA 的 Header。
   - 邻接关系建立后，路由器通过 DD 报文描述本端路由器的 LSDB，进行数据库同步。DD 报文里包括 LSDB 中每一条 LSA 的 Header ，即所有 LSA 的摘要信息。对端路由器根据 LSA Header 就可以判断是否已有这条 LSA。
3. **LSR 报文：**
   - 交换完 DD 报文后，需要发送 LSR 报文（Link State Request packet）向对方请求更新 LSA，LSR 报文里包括所需要的 LSA 的摘要信息。
4. **LSU 报文：**
   - LSU 报文（Link State Update packet）用来向对端路由器发送其所需的 LSA 或泛洪本端更新的 LSA。其报文内容是多条完整的 LSA 的集合。
5. **LSAck 报文：**
   - 为了实现可靠性传输，需要 LSAck（Link State Acknowledgment packet）用来对接到的 LSU 报文进行确认。
   - LSAck 报文的内容是需要确认的 LSA 的 Header，一个 LSAck 报文可对多个 LSA 进行确认。


####	DR 和 BDR 选举：

**Router ID：**

1. 如果两台路由器的 DR 优先级相等，需要进一步比较路由器的 Router ID，Router ID 大的将被选为 DR 或 BDR。
2. OSPF 的 Router ID 选取方式有两种：手动配置和设备自动选取。
3. 如果没有手动配置，设备会选取全局的 Router ID 作为 OSPF 的 Router ID。

以下 3 种情况会进行 Router ID 的重新选取：

- 通过 `ospf router-id xxx` 重新配置 OSPF 的 Router ID，并重启 OSPF 进程。
- 重新配置全局 Router ID，并重启 OSPF 进程。
- 原来被选取为全局 Router ID 的 IP 地址被删除，并重启 OSPF 进程。

**DR 和 BDR 选举的原因：**

在广播网络和NBMA网络中，如果任意两台路由器之间都要传递路由信息，即完全无向图中边的个数为（n * (n -1)） / 2 。任何一台路由器的路由变化都会导致多次传递，浪费了带宽资源。

为了解决这一问题，OSPF 定义了 DR，通过选举产生 DR 后，所有其他设备都只将信息发送给 DR，由 DR 将网络链路状态 LSA 广播出去。

为了防止 DR 发生故障，重新选举 DR 时会造成业务中断，除了 DR 之外，还会选举一个备份指定路由器 BDR。这样除 DR 和 BDR 之外的路由器（称为 DR Other）之间不再建立邻接关系，也不再交换任何路由信息，这样就减少了广播网和 NBMA 网络上各路由器之间邻接关系的数量。

**DR 和 BDR 选举的原则：**

OSPF 规定了一系列的选举规则：选举制，终身制，继承制。

1. 选举制：

   路由器接口的 DR 优先级决定了该接口在选举 DR，BDR 时所具有的资格，本网段内 DR 优先级大于 0 的路由器都可作为候选人。选举中使用的选票就是 Hello 报文，每台路由器将自己选出的 DR 写入 Hello 报文中，发给网段上的其他路由器。当处于同一网段的两台路由器同时宣布自己是 DR 时，DR 优先级高者胜出。如果优先级相等，则 Router ID 大者胜出。如果一台路由器的优先级为 0，则它不会被选举为 DR 或 BDR。

2. 终身制：

   终身制也称非抢占制。每一台新加入的路由器不急于参加选举，而是先考察一下本网段中是否已存在 DR。如果已有 DR，即使本路由器的 DR 优先级比现有的 DR 还高，也不会再声称自己是 DR。因为网段中的每台路由器都只和 DR，BDR 建立邻接关系，如果 DR 频繁更换，会导致短时间内网段中有大量的 OSPF 报文再传输，降低了可用带宽。终身制有利于增加网络的稳定性，提高网络的可用带宽。

3. 继承制：

   如果 DR 发生故障，且存在 BDR，那么下一个当选 DR 的一定是 BDR。其他路由器只能去竞选 BDR。这个原则可以保证 DR 的稳定性，因为 BDR 和 DR 的数据库是完全同步的，且和其他路由器建立了邻接关系，所以从角色切换到承载业务的时间会很短。当然后续还会选举一个新的 BDR，但是已经不会影响当前路由的计算了。



####	OSPF 接口状态机

OSPF 接口共有以下七种状态：

- Down：接口的初始状态。
- Loopback：设备到网络的接口处于环回状态，不能用于正常的数据传输，但可以通过 Router LSA 进行通告。
- Waiting：设备正在判定网络上的 DR 和 BDR。在设备参与 DR 和 BDR 选举之前，接口上会启动 Waiting 定时器（40秒）。在这个定时器超时前，设备发送的 Hello 报文不包含 DR 和 BDR 信息，设备不能被选举为 DR 或 BDR。
- P-2-P：仅 P2P，P2MP 网络有此状态。
- DROther：
- BDR：
- DR：



####	OSPF 邻居状态机

OSPF 网络中，相邻设备间通过不同的邻居状态切换，最终可以形成完全的邻接关系，完成 LSA 信息的交互。

OSPF 邻居共有以下八种状态：

- Down：邻居会话的初始状态，表明没有收到邻居设备的 Hello 报文。
- Attempt：适用于 NBMA 网络。
- Init：表示已经收到了邻居的 Hello 报文，但是对端并没有收到本端发送的 Hello 报文。对端的邻居列表中没有包含本端的 Router ID，双向通信仍热没有建立。
- 2 - Way：互为邻居。本状态标识双方相互都收到了对端发送的 Hello 报文，如果不形成邻接关系则状态机就停留在此状态，否则进入 ExStart 状态。DR 和 BDR 只有在邻居状态处于 2 - Way 及之后的状态才会被选举出来。
- ExStart：协商主从关系。建立主从关系主要是为了保证后续的 DD 报文交换中能够有序的发送。邻居间从此时才开始正式建立领接关系。

- Exchange：交换 DD 报文。本端设备将本地的 LSDB 用 DD 报文来描述，并发给邻居设备。
- Loading：正在同步 LSDB，发送 LSR 报文向邻居请求对方的 LSA，同步 LSDB。
- Full：建立邻接。两端设备的 LSDB 已同步，建立了完全的领接关系。 



####	OSPF 区域

骨干路由器：

区域边界路由器：

自治系统边界路由器：



**区域类型：**

1. **普通区域：**

   - 普通区域由标准区域和骨干区域组成。骨干区域由 Area 0 表示。骨干区域自身必须保证连通，所有非骨干区域必须与骨干区域保持连通。

2. **Stub 区域：**

   - Stub 区域的 ABR 不传播它们接收到的 AS 外部路由。Stub 区域是可选配置属性，一般位于 AS 的边界，是只有一个 ABR 的非骨干区域。


   - 不允许 Type 4，5 LSA 进入该区域，ABR 会自动下发一条 Type 3 缺省路由。
   - 骨干区域不能配置为 Stub 区域。
   - Stub 区域内不能存在 ASBR，因此 AS 外部的路由不能在本区域内传播。
   - 虚连接不能穿过 Stub 区域。

3. **Totally Stub 区域：**

   - 不允许 Type 3，4，5 LSA 进入该区域，ABR 会自动下发一条 Type 3 缺省路由。


   - 和 Stub 区域相比，Totally Stub 更彻底，拒绝 3 类 LSA 进入，只剩下唯一一条由 ABR 通告的缺省 Type 3 LSA 路由。 

![img](http://112.74.164.59//static/uploads/2024.10.25-16.00_b6221be3.png)

4. **NSSA 区域：**
   - NSSA（Not So Stubby Area）是 Stub 区域的一个变形，NSAA 区域不允许存在 Type 5 LSA。
   - NSAA 区域允许引入自治系统外部路由，携带这些路由信息的 Type 7 LSA 由 NSSA的 ASBR 产生，仅在本 NSSA 内传播。
     - 当 Type 7 LSA 到达 NSSA 的 ABR 时，由 ABR 将 Type 7 LSA 转换成 Type 5 LSA，泛洪到整个 OSPF 区域中。
     - NSSA 区域的 ABR 会发布 Type 7 LSA 缺省路由传播到本区域内。
   
5. **Totally NSSA 区域：**

   - 不允许发布 AS 外部路由和区域间路由，只允许发布区域内路由。

###	LSA 类型:



####	Type 1：Router - LSA

- 发布者：每一台路由器。
- 传播范围：仅在路由器所属区域内传播。
- 作用：描述链路状态和开销。



####	Type 2：Network - LSA

- 发布者：DR。
- 传播范围：仅在 DR 所属区域内传播。
- 作用：描述本网段的链路状态，列出和 DR 形成完全邻接关系的路由器的 Router ID。



通过 Router-LSA 和 Network-LSA 在区域内泛洪，区域内每个路由器可以完成 LSDB 同步，这就解决了区域内部的通信问题。



####	Type 3：Network - summary - LSA

- 发布者：ABR。
- 传播范围：ABR 交错传播给所属区域。
- 作用：描述区域间的路由信息，通告该区域到其他区域的目的地址，ABR 是将区域内部的 Type 1 和 Type  2 的信息收集起来并汇总之后扩散出去，这就是 Summary 的含义。



####	Type 4：ASBR - Summary - LSA

- 发布者：ABR。
- 传播范围：除了 ASBR 所在区域的其他相关区域。
- 作用：描述到 ASBR 的路由信息。



####	Type 5：AS - external - LSA

- 发布者：ASBR。
- 传播范围：除了 Stub 区域和 NSSA 区域以外的所有区域。
- 作用：描述到 AS 外部的路由。



####	Type 7：NSSA - LSA

- 发布者：ASBR。
- 传播范围：仅在 NSSA 区域内。
- 作用：描述到 AS 外部的路由，NSSA 区域的 ABR 收到 NSSA LSA 时，会有选择的将其转化为 Type 5 LSA，以便将外部路由信息通告到 OSPF 网络的其他区域。



####	Type 9，10，11：Opaque - LSA

通用扩展机制





###	OSPF 区域间环路及防环方法

OSPF 在区域内部运行的是 SPF 算法，这个算法能够保证区域内部的路由不会成环，但是在划分区域后，区域之间的路由传递实际上是一种类似距离矢量算法的方式，这种方式容易产生环路。

为了避免区域间的环路，OSPF 规定直接在两个非骨干区域之间发布路由信息是不被允许的，只允许在一个区域内部或者在骨干区域和非骨干区域之间发布路由信息。因此，每个 ABR 都必须连接到骨干区域。

总结就是，通过禁止非骨干区域间传递路由，避免了环路的产生。



###	OSPF 缺省路由





###	OSPF 虚连接

虚连接（Virtual link）是指在两台 ABR 之间通过一个非骨干区域建立一条逻辑上的连接通道。

由于在部署 OSPF 时，要求所有非骨干区域与骨干区域相连。但在实际应用中，可能会因为各方面条件的限制无法满足，此时可以通过配置 OSPF 虚连接来解决这个问题。

虚连接的存在增加了网络的复杂程度，而且排障困难。因此在网络规划中应该尽量避免使用虚连接，虚连接只是一种临时手段。虚链路可以看作一个表明网络中某个部分是否需要重新规划设计的标志。



###	OSPF 路由聚合和路由过滤

#### OSPF 路由聚合

ABR 可以将具有相同前缀的路由信息聚合在一起，只发布一条路由到其他区域。

通过路由聚合，可以减少路由信息，减小路由表的规模，提高设备的性能。

OSPF 有两种路由聚合方式：

- 区域间路由聚合
- 外部路由聚合

####	OSPF 路由过滤

OSPF 支持使用路由策略对路由信息进行过滤。缺省情况 OSPF 不进行过滤。

OSPF 可以使用 route-policy，访问控制列表（access-list）和地址前缀列表（prefix-list）。





###	命令示例



```bash
<R1> display ospf peer	# 查看 OSPF 邻居信息
<R1> display ospf 1 peer brief	# 查看 OSPF 邻居摘要信息
<R1> display ospf lsdb	# 查看 LSDB 信息，会显示所有 LSA
```





##	OSPF 高级特性-快速收敛



###	OSPF 快速收敛

OSPF 快速收敛扩展特性包括：

- I-SPF （Incremental SPF）

  增量最短路径优先算法，当网络拓扑改变的时候，只对受影响的节点进行路由计算，而不是对全部节点重新进行路由计算，从而加快了路由的计算。

- PRC （Partial Route Calculation）

  部分路由计算，是指当网络上路由发生变化的时候，只对发生变化的路由进行重新计算。不同的是，PRC不需要计算节点路径，而是根据 I-SPF 算出来的 SPT 来更新路由。

- 智能定时器

  OSPF 智能定时器可以根据用户的配置和触发事件（如路由计算）的频率动态调整时间间隔，使网络快速稳定。

总结：

I-SPF 来更新 最短路径树，PRC 来更新路由；PRC 和 I-SPF 配合使用是原始 SPF 算法的改进，所以已经代替了原有的算法。智能定时器在不稳定或是频繁进行路由计算的网络中，根据配置可以动态调整计算路由或泛洪 LSA 的时间间隔，加速了网络收敛，缺省下已经开启并配置了缺省值。

















###	OSPF IP FRR

OSPF FRR 利用 LFA（Loop-Free Alternates）算法预先计算出备份路径，保存在转发表中，以备在故障时将流量快速切换到备份链路上，保证流量不中断，该功能可将故障恢复时间降至 50ms 以内。



LFA 计算备份链路的基本思路是：

- 以可提供备份链路的邻居为根节点，利用 SPF 算法计算出到目的节点的最短距离。然后按照不等式计算出开销最小的无环的备份链路。



####	OSPF IP FRR

OSPF IP FRR 的流量保护分为链路保护和节点链路双保护。

节点保护的优先级高于链路保护。



####	基础配置命令

1. 使能 OSPF IP FRR：

   ```bash
   [AR-ospf-1]: frr
   [AR-ospf-1-frr]: loop-free-alternate
   ```

   利用 LFA 算法计算备份下一跳和备份出接口。

2. 可选：阻止 OSPF 接口的 FRR 能力

   ```bash
   [AR-G0/0/1]: ospf frr block
   ```

   





###	OSPF 和 BFD 联动

将 BFD 和 OSPF 关联起来，BFD 对链路故障的快速感应能够加快 OSPF 对于网络拓扑变化的响应。

**联动的工作原理：**

- 邻居状态机到达 Full 时，通知 BFD 建立 BFD 会话。



**OSPF 与 BFD 联动的基础配置命令：**

1. 配置 OSPF 的 BFD 特性：

   ```shell
   # 在 OSPF 进程下使能 BFD 特性
   [AR-ospf-1] bfd all-interfaces enable
   
   # 配置 BFD 会话的参数值
   [AR-ospf-1] bfd all-interfaces {min-rx-interval xx | min-tx-interval xx | detect-multiplier xx | frr-binding}
   
   
   ```

   - min-rx-interval: 指定期望从对端接收 BFD 报文的最小接收间隔
   - min-tx-interval: 指定向对端发送 BFD 报文的最小发送间隔
   - detect-multiplier: 指定本地检测倍数
   - frr-binding: 将 BFD 会话状态和接口的链路状态进行绑定，当 BFD 会话状态变为 Down 时，接口的物理层链路状态也会变为 Down，从而触发流量切换到备份路径。

2. 配置指定接口的 BFD 特性：

   ```shell
   # 在开启 OSPF 的接口下使能 BFD
   [AR-G0/0/0] opsf bfd enable
   
   ```

   





##	OSPF 高级特性-路由控制

OSPF 路由控制包括：

- 调整 OSPF 的接口开销
- 设置等价路由
- 引入外部路由
- 路由聚合
- 缺省路由通告
- Filter-Policy
- 对发送的 LSA 进行过滤
- 设置 LSDB 中 External LSA 的最大数量



####	等价路由

当路由表中存在到达同一目的地址，且同一路由协议发现的多条路由时，若这几条路由的开销值也相同，那么这些路由就是等价路由，可以实现负载分担。

设置进行负载分担的等价路由的最大数量：

```bash
[AR-ospf-1] maximum load-balancing xx
```



####	缺省路由

OSPF 缺省路由通常应用于下面两种情况：

- 由区域边界路由器 ABR 发布 Type3 LSA，用来指导区域内路由器进行区域之间报文的转发。
- 由自治系统边界路由器 ASBR 发布 Type 5，Type 7 LSA，用来指导 OSPF 路由域内路由器进行域外报文的转发。

|       区域类型       |                 产生条件                 | 发布方式 | 产生 LSA 类型 | 泛洪范围  |
| :------------------: | :--------------------------------------: | :------: | :-----------: | :-------: |
|       普通区域       |    通过 default-route-advertise 配置     |   ASBR   |   Type5 LSA   | 普通区域  |
| Stub 和 Totally Stub |                 自动产生                 |   ABR    |   Type3 LSA   | Stub 区域 |
|      NSSA 区域       | 通过 nssa [default-route-advertise] 配置 |   ASBR   |   Type7 LSA   | NSSA 区域 |
|  Totally NSSA 区域   |                 自动产生                 |   ABR    |   Type3 LSA   | NSSA 区域 |
|                      |                                          |          |               |           |

OSPF 路由的分级管理，Type3 缺省路由的优先级高于 Type5 或 Type7 路由。

- 普通区域：
  - 缺省情况下，普通区域内的路由器不会产生缺省路由，即使它有缺省路由。当路由器需要向 OSPF 发布缺省路由时，必须手工执行 default-route-advertise 命令，配置完成后，路由器会产生一个缺省 ASE LSA （Type 5），并且通告到整个 OSPF AS 中。
- Stub 区域：
  - Stub 区域不允许 AS 外部的路由（Type5 LSA）在区域内传播。区域内的路由器必须通过 ABR 学习到 AS 外部的路由。
  - Stub 区域的 ABR 会自动产生一条缺省的 Type3 LSA 通告到整个 Stub 区域。ABR 通过该缺省路由，将到达 AS 外部的流量吸引到自己这里。
- Totally Stub 区域：
  - 和 Stub 区域相比，就多了不允许区域间路由（Type3 LSA）在区域内传播。区域间的路由器必须通过 ABR 学习到 AS 外部和区域间的路由。同样会自动产生一条缺省的 Type3 LSA 通告到整个 Stub 区域。
- NSSA 区域：
  - NSSA 区域允许 Type7 LSA 的传播，Type7 LSA 由 NSSA 区域的 ASBR 产生，当它到达 NSSA 区域的 ABR 时，可以由 ABR 转换成 Type5 LSA，并通告到其他区域。
  - 
- Totally NSSA 区域：



####	将缺省路由通告到 OSPF 路由区域

1. 通告到普通区域：

   ```bash
   # 将缺省路由通告到普通 OSPF 区域
   [AR-ospf-1] default-route-advertise [always|permit-calculate-other] | cost xx | type xx | route-policy xx
   
   # 指定 Type3 Summary-LSA 的缺省开销值
   [AR-ospf-1] defalut-route-advertise summary cost xx
   ```

   注意，import-route （OSPF）命令不能引入外部路由的缺省路由，当需要引入其他协议产生的缺省路由时，必须在 ASBR 上配置 default-route-advertise 命令，发布缺省路由到整个普通 OSPF 区域。

   - always：无论本机是否存在非 OSPF 缺省路由，都会产生并发布一个描述缺省路由的 LSA。
   - type：第一类外部路由还是第二类外部路由。
   - 





####	对发送的 LSA 进行过滤

当两台路由器之间存在多条链路时，可以在某些链路上通过对发送的 LSA 进行过滤，减少不必要的重传，节省带宽资源。

配置对 OSPF 接口出方向的 LSA 进行过滤：

```bash
[AR-G0/0/0] ospf filter-lsa-out {}
```



OSPF Database Overflow 概述

通过设置 LSDB 中 External LSA （Type5, 7）的最大条目数，可以动态限制数据库的规模。

```bash
[AR-ospf-1] lsdb-overflow-limit xx
```

![image-20241030171236986](C:\Users\46823\AppData\Roaming\Typora\typora-user-images\image-20241030171236986.png)



