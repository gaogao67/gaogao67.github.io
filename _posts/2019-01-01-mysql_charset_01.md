---
layout: post
title:  MySQL字符集基础
date:   2019-01-16 15:00:00 +0800
categories: MySQL-CharSet
tag: MySQL-CharSet
---

* content
{:toc}


字符集和编码规则
====================================
```

字符集：特定类型或用途的字符集合，为集合中每个"字符"分配一个唯一的ID值(Code Point)

编码规则：将字符集中的字符按照其对应的ID值转换为字节系列的规则(编码和解码)
UNICODE是字符集，而UTF和UTF16以及UTF32属于编码规则。

ASCII编码：American Standard Code for Information Interchange，美国信息互换标准代码，使用1字节8Bit来存放2的8次方255个字符。
GB2312：在ASCII编码的基础上进行扩展，收录6000+常用汉字，形成GB2312编码
GBK：在GB2312的基础上扩展，收录繁体和不常用汉字以及各种字符，形成GBK编码
GB18030：在GBK的基础上扩展，收录各名字的独立的字符，最终形成GB18030
UNICODE： 全球标准化组织ISO统一对全球各种文字和字符进行编码形成的字符集。
UTF8：基于UNICODE字符集，以8Bit为一个编码单位的可变长编码规则，最新UTF8将每个Unicode字符编码为1到4字节长度。
UTF16：基于UNICODE字符集，以16Bit为一个编码单位的可变长编码规则,最新UTF16将每个Unicode字符编码为2个或4个字节长度。


```

Unicode字符集与UTF-8编码
=======================================
```

Unicode编码与UTF-8的编码的对应关系:
Unicode编码 UTF-8编码(二进制)
U+0000 		– U+007F 		0xxxxxxx
U+0080 		– U+07FF 		110xxxxx 10xxxxxx
U+0800 		– U+FFFF 		1110xxxx 10xxxxxx 10xxxxxx
U+10000 	– U+10FFFF 		11110xxx 10xxxxxx 10xxxxxx 10xxxxxx

一个字节的uft8表示的unicode 码范围为(0 ~0x7F)
两个字节长度的uft8 表示的unicode码范围为(0x80 ~ 0x07FF)
三个字节长度的uft8 表示的unicode码范围为(0x0800 ~ 0xFFFF)
四个字节长度的uft8 表示的unicode码范围为( 0x10000 ~ 0x10FFFF)

```


MySQL的UTF8和UTF8MB4
========================================
```

MySQL 与UTF8
在MySQL刚开发时，Unicode字符集还不够完善，MySQL使用只支持最长三字节的UTF8字符集便可以存放所有Unicode字符，随着Unicode的完善，Unicode字符集收录的字符数量越来越多，最新版本的UTF8需要使用1到4个字节来存放Unicode字符,但MySQL为保持版本兼容，依旧使用最多3字节的UTF8字符集，因此MySQL中的UTF8不能完全兼容最新的UTF8版本，导致很多生僻字符和网络表情无法正常存储到MySQL中。

为兼容最新的UTF8字符集，在MySQL 5.5.3版本引入UTF8MB4字符集。

```


MySQL字符集查看和修改
==========================================
```sql
#改表的字符集为UTF8
ALTER TABLE TBName CONVERT TO CHARACTER SET utf8;

#看字符集对应的默认排序规则
SHOW CHARACTER SET LIKE 'utf8' \G

#查看当前字符编码的设置
SHOW VARIABLES LIKE 'character_set%' \G

#查看每列的字符集
SHOW FULL COLUMNS FROM TB_Name \G

```