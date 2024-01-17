---
title: "Encoding"
categories:
  - Blog
tags:
  - UTF-8
  - Terminal
last_modified_at: 2024-1-17
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

## linux 下的不同

- 在 linux bash 是没有 powershell 的 inputencoding 选项的, 其默认是与本地 locale 相关, 一般都为\*\*.UTF-8, 前缀一般是 en_US, zh_CN, 表示地区

- 所以如果要显示 cp936 编码的文件, 就要格式转换, 使用 iconv

```bash
cat somefile | iconv -f cp936 -t utf-8
```

- 对于文件名, 则需要不同的处理, 因为 ntfs 默认编码为 utf-16, linux 默认为 utf-8, 一个 cp936 的文件, 在 ls 从文件系统读取文件名时就会乱码, 然后会输出乱码, 导致 iconv 出错, 我们必须想办法得到原来的二进制文件名

- linux 下有 convmv 命令专门处理文件名, 或者用 python, 传入二进制路径得到二进制文件名从而 encoding 到正确的编码, 代码如下

```bash
convmv --notest -f cp936 -t utf-8 *.txt
```

```python
for path in os.listdir(b'path'):
  print(path.decode(encoding='cp936'))
```

## 几种乱码形式

- 不兼容的字符类型很可能会导致信息丢失, 使其无法还原

1. 出现古文夹杂日韩文，以 GBK 读取 UTF-8 编码
2. 出现方块形，以 UTF-8 读取 GBK
3. 各种符号，以 ISO8859-1 方式读取 UTF-8
4. 拼音码，带声调的字母，以 ISO8859-1 方式读取 GBK
5. 长度为偶数正常, 长度为奇数时，最后的字符变成问号，以 GBK 读取 UTF-8 编码，再用 UTF-8 格式再次读取。
6. 大部分文字为锟斤拷，以 UTF-8 方式读取 GBK 码，再次用 GBK 格式再次读取
