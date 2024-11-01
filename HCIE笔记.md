Vlanif 接口：

- 三层交换机上的一个逻辑接口。
- Vlanif ID 对应的是 vlan。
- Vlanif 是 vlan 的 网关接口（三层接口）。Vlanif 接口一定有 mac 地址。
- Vlanif 接口 Up 的必要条件是：
  - Vlanif 接口对应的 vlan，必须已经创建。
  - 必须有 Up 的物理接口或 Trunk 接口已经加入 Vlanif 对应的 vlan。

VLAN：

- PVID 一般不变更，极少数情况下需要变更。
- Vlanif 终结掉数据帧的 Tag，添加所属 VLAN Tag 转发？