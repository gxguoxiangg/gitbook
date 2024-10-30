---
title: 系统封装
date: 2024-10-08 23:48:00
tags:
    - 笔记
    - Windows
---



##	封装步骤

1. 下载目标原始 Windows 镜像，提取里面的 install.wim 文件。
2. （可选）使用 NTLite 进行定制，可精简 Windows Denfer 等系统组件。
3. 安装 install.wim，开机进入系统按 Ctrl + Shift + F3 跳过 OOBE 进入审核模式，检查系统状态。
4. 根据 checklist ，安装常用软件，微软 C++ 运行库等，启动 ES5 进行第一阶段封装，完成后重启进入 PE。
5. 重启进入 PE 后，此时会多出 C:\Sysprep 目录。将万能驱动，激活脚本放入此目录中。启动 ES5 进入第二阶段封装。完成后将在指定路径下生成 wim 文件。
6. 使用 wim 文件安装部署即可。进入桌面等待部署完成后系统将自动删除 C:\Sysprep 目录。





NTLite 相关设置参考：https://blog.csdn.net/itfans123/article/details/135266224

ES5 工具的相关设置参考：

- https://blog.csdn.net/hrb880/article/details/126963106

- https://blog.csdn.net/weixin_43483442/article/details/137053156





##	注意事项

1. 母盘的版本可能影响封装后的部署。

   使用 LSTC 2019 进行封装后，部署时卡在 70% 很久，规避了封装系统的初衷。使用 LSTC 2021 作为母盘则无任何问题。

2. 





##	资源列表

- NTLite：[https://www.ntlite.com/download/]
- EasyDrv7 万能驱动：[https://pan.baidu.com/share/init?surl=Gs6uY2u6Xn5A-4NjSqt60Q&pwd=gw5u]，提取码：gw5u
- Easy Sysprep 5 Plus（Beta 10，2024.9.23 发布）：[https://www.itsk.com/thread/432571]，[https://www.alipan.com/s/SpuuFUhgGt6]





## 备份录

使用 virtual box 启动刚装好系统的虚拟磁盘报错：

```cmd
No bootable medium found.
Please insert a bootable medium and reboot
```

解决方法：https://blog.csdn.net/cowboyjisuanji/article/details/130862803