---
layout: post
title:  使用dmidecode看错硬件信息
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Hardware
tag: Hardware
---

* content
{:toc}


基础常识
====================================
```

DMI (Desktop Management Interface, DMI)就是帮助收集电脑系统信息的管理系统，DMI信息的收集必须在严格遵照SMBIOS规范的前提下进行。 SMBIOS(System Management BIOS)是主板或系统制造者以标准格式显示产品管理信息所需遵循的统一规范。SMBIOS和DMI是由行业指导机构Desktop Management Task Force (DMTF)起草的开放性的技术标准，其中DMI设计适用于任何的平台和操作系统。

DMI充当了管理工具和系统层之间接口的角色。它建立了标准的可管理系统更加方便了电脑厂商和用户对系统的了解。DMI的主要组成部分是Management Information Format (MIF)数据库。这个数据库包括了所有有关电脑系统和配件的信息。通过DMI，用户可以获取序列号、电脑厂商、串口信息以及其它系统配件信息。


```


常用命令
====================================
```

1、查看服务器型号：dmidecode | grep 'Product Name'
2、查看主板的序列号：dmidecode |grep 'Serial Number'
3、查看系统序列号：dmidecode -s system-serial-number
4、查看内存信息：dmidecode -t memory
5、查看OEM信息：dmidecode -t 11


```

命令详解
====================================
```
dmidecode不带选项执行会输出所有的硬件信息。

Dmidecode 有个很有用的选项 -t，可以按指定类型输出相关信息，假如要获得处理器方面的信息，则可以执行
dmidecode -t processor


Usage: dmidecode [OPTIONS]
Options are:
-d, --dev-mem FILE Read memory from device FILE (default: /dev/mem)
#从设备文件读信息，输出内容与不加参数标准输出相同
-h, --help Display this help text and exit
#显示帮助信息
-q, --quiet Less verbose output
#显示更少的简化信息
-s, --string KEYWORD Only display the value of the given DMI string
#只显示指定DMI字符串的信息
-t, --type TYPE Only display the entries of given type
#只显示指定条目的信息
-u, --dump Do not decode the entries
#显示未解码的原始条目内容
--dump-bin FILE Dump the DMI data to a binary file
--from-dump FILE Read the DMI data from a binary file
-V, --version Display the version and exit
#显示版本信息


dmidecode参数string及type列表

（1）Valid string keywords are:
bios-vendor
bios-version
bios-release-date
system-manufacturer
system-product-name
system-version
system-serial-number
system-uuid
baseboard-manufacturer
baseboard-product-name
baseboard-version
baseboard-serial-number
baseboard-asset-tag
chassis-manufacturer
chassis-type
chassis-version
chassis-serial-number
chassis-asset-tag
processor-family
processor-manufacturer
processor-version
processor-frequency

（2）Valid type keywords are:
bios
system
baseboard
chassis
processor
memory
cache
connector
slot

（3）type全部编码列表
Type Information
—————————————-
0 BIOS
1 System
2 Base Board
3 Chassis
4 Processor
5 Memory Controller
6 Memory Module
7 Cache
8 Port Connector
9 System Slots
10 On Board Devices
11 OEM Strings
12 System Configuration Options
13 BIOS Language
14 Group Associations
15 System Event Log
16 Physical Memory Array
17 Memory Device
18 32-bit Memory Error
19 Memory Array Mapped Address
20 Memory Device Mapped Address
21 Built-in Pointing Device
22 Portable Battery
23 System Reset
24 Hardware Security
25 System Power Controls
26 Voltage Probe
27 Cooling Device
28 Temperature Probe
29 Electrical Current Probe
30 Out-of-band Remote Access
31 Boot Integrity Services
32 System Boot
33 64-bit Memory Error
34 Management Device
35 Management Device Component
36 Management Device Threshold Data
37 Memory Channel
38 IPMI Device
39 Power Supply
40 Additional Information
41 Onboard Device


```


