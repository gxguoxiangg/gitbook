####	使用 SNMP 监控网络设备

大致步骤：

1. 编写自定义的 *yml* 文件，使用 *generator* 生成 `snmp.yml`。
2. 使用 `snmp.yml`，运行 *snmp_exporter.exe*。
3. 运行 *prometheus*。
4. 运行 `grafana.exe`



####	1.Generator 生产 snmp.yml

自定义的 *yml* 样例如下：

```yaml
auths:
  auth_h3c: # 认证模块名
    version: 2
    community: netinfo_ro_guoxiang

modules:
  h3c_common: # 指标模块名
    walk:
      # 交换机基础信息
      - 1.3.6.1.2.1.1.1                       # sysDescr - 设备描述，用于识别设备型号和软件版本
      - 1.3.6.1.2.1.1.5                       # sysName - 系统名称
      - 1.3.6.1.2.1.1.3                       # sysUpTime - 设备上电时间
      - 1.3.6.1.2.1.1.6                       # sysLocation: 设备所在位置

      # 实体CPU和内存信息
      - 1.3.6.1.4.1.25506.2.6.1.1.1.1.6       # hh3cEntityExtCpuUsage - 实体 CPU 实时利用率
      - 1.3.6.1.4.1.25506.2.6.1.1.1.1.8       # hh3cEntityExtMemUsage - 实体内存实时利用率百分比
      # 实体风扇和电源状态信息
      - 1.3.6.1.2.1.47.1.1.1.1.5              # entPhysicalClass - 实体类型
      - 1.3.6.1.2.1.47.1.1.1.1.7              # entPhysicalName - 实体名称
      # prometheus 通过合并查询实现
      - 1.3.6.1.4.1.25506.2.6.1.1.1.1.19      # hh3cEntityExtErrorStatus - 实体错误状态
      # 实体传感器温度信息
      - 1.3.6.1.4.1.25506.2.6.1.1.1.1.12      # hh3cEntityExtTemperature - 实体温度
      # 电源实体当前功率
      - 1.3.6.1.4.1.25506.2.6.1.3.1.1.3       # hh3cEntityExtCurrentPower - 实体当前功率 单位是毫瓦
      - 1.3.6.1.4.1.25506.2.6.1.3.1.1.4       # hh3cEntityExtAveragePower - 实体平均功率 单位是毫瓦
      # 存储介质信息
      - 1.3.6.1.4.1.25506.2.5.1.1.4.1.1.4     # hh3cFlhPartSpace - 存储设备分区容量 单位byte
      - 1.3.6.1.4.1.25506.2.5.1.1.4.1.1.5     # hh3cFlhPartSpaceFree - 存储介质分区大小
      - 1.3.6.1.4.1.25506.2.5.1.1.4.1.1.10    # hh3cFlhPartName - 存储设备分区名称

    max_repetitions: 20
    retries: 3
    timeout: 5s

    lookups:
      # hh3cEntityExtPhysicalIndex = entPhysicalIndex
      - source_indexes: [entPhysicalIndex]
        lookup: entPhysicalClass
      - source_indexes: [entPhysicalIndex]
        lookup: entPhysicalName

    overrides:
      entPhysicalClass:
        ignore: true
  
  h3c_stack:
    walk:
      # 堆叠信息
      - 1.3.6.1.4.1.25506.2.91.1.7            # hh3cStackTopology - 堆叠系统的拓扑类型
      - 1.3.6.1.4.1.25506.2.91.1.2            # hh3cStackMemberNum - 本IRF系统目前包含的堆叠设备数量
      - 1.3.6.1.4.1.25506.2.91.4.1.1          # hh3cStackPortIndex 当前堆叠口在本设备上的逻辑编号
      - 1.3.6.1.4.1.25506.2.91.4.1.2          # hh3cStackPortEnable 当前堆叠接口是否使能
      - 1.3.6.1.4.1.25506.2.91.4.1.3          # hh3cStackPortStatus 当前堆叠口的链路状态
      - 1.3.6.1.4.1.25506.2.91.4.1.4          # hh3cStackNeighbor 与当前堆叠口相连的堆叠设备的设备成员编号
      
    max_repetitions: 20
    retries: 3
    timeout: 5s
    
    lookups:
      # h3cStackmemberID和h3cStackPortIndex
      - source_indexes: [h3cStackmemberID, h3cStackPortIndex]
        lookup: hh3cStackPortEnable
      - source_indexes: [h3cStackmemberID, h3cStackPortIndex]
        lookup: hh3cStackNeighbor

    overrides:
      hh3cStackPortEnable:
        ignore: true
      hh3cStackNeighbor:
        ignore: true


  h3c_interface:
    walk:
      # 接口信息 - 索引 ifIndex
      - 1.3.6.1.2.1.2.2.1.2                   # ifDescr - 接口描述
      - 1.3.6.1.2.1.31.1.1.1.18               # ifAlias - 接口别名
      - 1.3.6.1.2.1.31.1.1.1.1                # ifName - 接口名字
      - 1.3.6.1.2.1.31.1.1.1.15               # ifHighSpeed - 接口当前带宽
      - 1.3.6.1.2.1.2.2.1.7                   # ifAdminStatus - 接口默认状态
      - 1.3.6.1.2.1.2.2.1.8                   # ifOperStatus - 接口运行状态
      - 1.3.6.1.2.1.2.2.1.13                  # ifInDiscards - 入方向丢包统计
      - 1.3.6.1.2.1.2.2.1.14                  # ifInErrors - 入方向错包统计
      - 1.3.6.1.2.1.2.2.1.19                  # ifOutDiscards - 出方向丢包统计
      - 1.3.6.1.2.1.2.2.1.20                  # ifOutErrors - 出方向错包统计
      - 1.3.6.1.2.1.31.1.1.1.6                # ifHCInOctets - 入方向报文统计
      - 1.3.6.1.2.1.31.1.1.1.10               # ifHCOutOctets - 出方向报文统计


      # 光模块信息 - 索引 ifIndex
      - 1.3.6.1.4.1.25506.2.70.1.1.1.9        # hh3cTransceiverCurTXPower 光模块当前的发送光功率 单位为百分之一dBM
      - 1.3.6.1.4.1.25506.2.70.1.1.1.12       # hh3cTransceiverCurRXPower 光模块当前的接收功率 单位为百分之一dBM
      - 1.3.6.1.4.1.25506.2.70.1.1.1.15       # hh3cTransceiverTemperature 光模块当前的温度 单位为摄氏度
      - 1.3.6.1.4.1.25506.2.70.1.1.1.20       # hh3cTransceiverTempHiWarn 温度预警上限值，单位为千分之一摄氏度
      - 1.3.6.1.4.1.25506.2.70.1.1.1.32       # hh3cTransceiverPwrOutHiWarn 输出功率预警上限值 单位为十分之一微瓦 为0时代表不支持
      - 1.3.6.1.4.1.25506.2.70.1.1.1.33       # hh3cTransceiverPwrOutLoWarn 输出功率预警下限值,单位为十分之一微瓦
      - 1.3.6.1.4.1.25506.2.70.1.1.1.36       # hh3cTransceiverRcvPwrHiWarn 输入功率预警上限值,单位为十分之一微瓦
      - 1.3.6.1.4.1.25506.2.70.1.1.1.37       # hh3cTransceiverRcvPwrLoWarn 输入功率预警下限值,单位为十分之一微瓦 

    max_repetitions: 25
    retries: 3
    timeout: 5s
    
    lookups:
      - source_indexes: [ifIndex]
        lookup: ifDescr
      - source_indexes: [ifIndex]
        lookup: ifAlias
      - source_indexes: [ifIndex]
        lookup: ifName
      - source_indexes: [ifIndex]
        lookup: ifHighSpeed
      - source_indexes: [ifIndex]
        lookup: ifAdminStatus
      - source_indexes: [ifIndex]
        lookup: ifOperStatus
    
    overrides:
      ifDescr:
        ignore: true
      ifAlias:
        ignore: true
      ifName:
        ignore: true
      # ifHighSpeed:
      #   ignore: true
      ifAdminStatus:
        ignore: true
      ifOperStatus:
        ignore: true
    
    filters:
      dynamic:
        - oid: 1.3.6.1.2.1.2.2.1.7
          targets:
            - 1.3.6.1.2.1.2.2.1.8
            - 1.3.6.1.2.1.31.1.1.1.6
            - 1.3.6.1.2.1.31.1.1.1.10
            - 1.3.6.1.2.1.2.2.1.13
            - 1.3.6.1.2.1.2.2.1.19
            - 1.3.6.1.2.1.2.2.1.14
            - 1.3.6.1.2.1.2.2.1.20
            - 1.3.6.1.4.1.25506.2.70.1.1.1.9
            - 1.3.6.1.4.1.25506.2.70.1.1.1.12
            - 1.3.6.1.4.1.25506.2.70.1.1.1.15
            - 1.3.6.1.4.1.25506.2.70.1.1.1.20
            - 1.3.6.1.4.1.25506.2.70.1.1.1.32
            - 1.3.6.1.4.1.25506.2.70.1.1.1.33
            - 1.3.6.1.4.1.25506.2.70.1.1.1.36
            - 1.3.6.1.4.1.25506.2.70.1.1.1.37
          values: ["1"]
```

需要使用 Go 编译源码生成 generator.exe。这里建议在 Linux 环境进行编辑，Windows下太多依赖难以安装。

```bash
cd generator/
go build
```

生成好 generator.exe 后，下载 mib 文件放进指定目录中，再使用如下命令，生成 *snmp.yml*：

```ini
./generator --fail-on-parse-errors generate -m ./mibs/ -g ./gx.yml -o snmp_gx.yml
```

H3C MIB 下载：

ComwareV5：https://www.h3c.com/cn/d_200905/635750_473262_0.htm

ComwareV7：https://www.h3c.com/cn/d_201806/1089291_473262_0.htm

ComwareV9：https://www.h3c.com/cn/d_202310/1943062_473262_0.htm



####	运行 *snmp_exporter.exe*

将 *snmp.yml*，放进指定目录中，运行 snmp_exporter.exe：

```bash
cd snmp_exporter-0.29.0.windows-amd64/
./snmp——exporter.exe
```

此时，浏览器应该可以打开 http://localhost:9916，可以使用此链接查询测试： 

http://localhost:9116/snmp?target=10.34.32.1&auth=auth_h3c&module=h3c_common



####	编辑 *prometheus.yml* 后，运行 prometheus.exe

编辑 *prometheus.yml* ，结合自定义的 *yml* 文件，示例如下：

```yaml
global:
  scrape_timeout: 15s

scrape_configs:
  - job_name: "CORE"
    static_configs:
      - targets:
        - 10.34.32.1
        # - 10.34.40.2
    metrics_path: /snmp
    params:
      module: [h3c_common, h3c_interface, h3c_stack]
      # module: [h3c_common, h3c_interface, ]
      auth: [auth_h3c]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance  
      - target_label: __address__
        replacement: 127.0.0.1:9116  # The SNMP exporter's real hostname:port.
```



