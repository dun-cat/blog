---
layout: post
title: 命令 ：grep
date: 2017-12-19
tags: ["grep","Linux","命令"]
---

###  grep

文字检索工具。

#### 使用
``` bash
grep anytext demo.txt # 在demo.txt查找匹配字符

grep 'hello world' demo.txt # 字符串匹配

grep -i anytext demo.txt  # 忽略大小写

grep -v # 反选

grep -c # 打印已匹配的个数
```
    