---
title: "Encoding"
categories:
  - Blog
tags:
  - UTF-8
  - Terminal
last_modified_at: 2024-1-11
---

# Encoding

编码问题一直是在 windows 下开发一直老大难的问题, 经常遇到各种乱七八糟的问题, 本 blog 主要记录一些解决方案

## 终端字体呈现

终端字体从程序里"输出"到屏幕上, 经历数个过程, 每一个过程其实都有对应的编码问题

- 源文件编码, 也就是源代码的编码, 一般都是 UTF-8, gbk, gcc 可以通过 `-finput-charset=utf-8` 来指定源文件编码, msvc 可以通过 `/source-character-set:gbk` 来指定源文件编码, 保证其 flag 与你源码编码格式相同即可

- 编译程序的内部编码, windows 下 visual studio 一般是 gbk, vscode 默认是 utf-8, 其他平台一般是 utf-8, gcc 可以通过 `-fexec-charset=gbk` 来指定, msvc 可以通过 `/execution-charset:utf-8` 来指定

- shell 内部编码, shell 接受程序输出字符, 并传送给终端, powershell 可以设置其如何处理程序的输出, 并输出到终端, 如下 powershell 命令就设置了其处理程序输出的时编码

```powershell
[console]::InputEncoding = New-Object System.Text.UTF8Encoding
```

- 最后就是终端编码, 终端接受 shell 输出的字符, 并显示到屏幕上, 如下 powershell 命令可以设置 powershell 向终端输出的编码

```powershell
$OutputEncoding = [console]::OutputEncoding = New-Object System.Text.UTF8Encoding
```

- 去除乱码的关键就是这四步, 保证这四次编码相邻两次格式一致就可以不出现乱码

- 比如源文件是 gbk, 加入`-finput-charset=gbk,-fexec-charset=utf-8`, 保证编译程序的内部编码就是正确的 utf-8,

- 设置 powershell InputEncodng 为 utf-8, OutputEncoding 为 gbk, powershell 就能正确处理程序输出的 utf-8 字符, 并传送给终端 gbk 字符

- 最后终端编码如果是 gbk, 与 powershell 输出一致, 如此就不会出现乱码,

- 当然最简单就是保证他们全部都是 utf-8, 最省心, 在 linux 其实就是这样
