---
layout: post
title:  使用dmesg用来显示开机信息
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Hardware
tag: Hardware
---

* content
{:toc}


基础常识
====================================
```

Linux命令dmesg用来显示开机信息，kernel会将开机信息存储在ring buffer中。在系统引导时，内核将与硬件和模块初始化相关的信息填到这个缓冲区中。内核环缓冲区中的消息对于诊断系统问题 通常非常有用。您若是开机时来不及查看信息，可利用dmesg来查看。开机信息亦保存在/var/log目录中，名称为dmesg的文件里。

```


命令使用
====================================
```

语　　法：dmesg [-cn][-s <缓冲区大小>]
参　　数：
　-c 　显示信息后，清除ring buffer中的内容。 
　-s<缓冲区大小> 　预设置为8196，刚好等于ring buffer的大小。 
　-n 　设置记录信息的层级。


```

参考链接：http://blog.csdn.net/zhongyhc/article/details/8909905