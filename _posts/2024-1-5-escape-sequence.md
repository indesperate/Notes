---
title: "Escape sequence"
categories:
  - Blog
tags:
  - Bash
  - Terminal
last_modified_at: 2023-11-30
---

# Escape Sequence

Escape sequences 是一种用于编程和数据通信中的特殊字符序列, 用于表示那些不能直接在代码或文本中表示的控制字符或特殊字符.

[wiki](https://en.wikipedia.org/wiki/ANSI_escape_code)上介绍的很详细, 特殊的转义字符可以完成很多好玩的事情, 比如改变 font, font color, cursor, 可以用来执行一些 system control.

## Escape Sequences type

1. 直接表示 ASCII characters

- `\n` (newline), `\t` (tab), `\\` (backslash), `\a` (bell), `\b` (backspace). 通过 `man ASCII` 可以直接查看, 这里的 n t 为简写, 实际上可以用不同进制的数来写

2. ANSI Escape Sequences

- 以`<ESC>`的 ASCII 码为开头, 可以为`\033`(8 进制) or `\x1b`(16 进制) or `\e`, 接相应的字符来表示不同的功能, `<ESC>`在 vim 里显示为`^[`

- powershell 的默认转义符号不为`\`, 为`, 因为 windows 下这玩意是路径分隔符

- `<ESC>\` 或者`\x9c` 一般表示为 ST, 为 String terminator, 但是现在`\a`, 也就是 bell 也可以为此作用, [stackoverflow](https://unix.stackexchange.com/questions/208436/bell-and-escape-character-in-prompt-string)上写了这个来由, 这个不是标准行为, 但大家都这么用

## Control Sequence Introducer (CSI)

1. CSI 格式以`<ESC>`的 ASCII 开头, 接`[`符号, 这种特殊的格式便可以用来定义 font, cursor 等, Wiki 中给出了一些 squeunce 例子

- CSI = `\e[`

2. cursor scroll 相关

- `<CSI>nA`, Cursor UP n line, `<CSI>nE` Cursor n Next line,
  `<CSI>n;mH` cursor position row n, column m
  `<CSI>nS` Scroll Up n line

- 简单的例子

```bash
echo -ne "\e[1;10H"
```

3. 此外还有 Hide Cursor, report cursor position

4. 另外一个比较常用是`<CSI>nm` 用来改变文字的格式, 也就是 SGR

### SGR(Selcet Graphic Rendition), `<CSI>nm`

这里选择几个比较常见的 SGR, 具体可以见 wiki

1. `<CSI>0m`, 关闭所有设置

2. n 从 1-29 都是一些 style 设置,30 开始至 49 都是设置颜色, 往后又是一些 style 设置, 至 90 开始是亮色的设置

   - 颜色 FG(foreground) 范围在 30-37, 90-97(bright color), BG(background) 40-47, 100-107(bright color)

3. - 以上都是 3-bit or 4-bit 色, 8bit 色便有自己独特的格式 `<ESC>[38;5;nm`(FG) `<ESC>[48;5;nm`(BG)

- 此处的 n 可以为以下值
  - 0-7: standard colors (as in ESC [ 30–37 m)
  - 8-15: high intensity colors (as in ESC [ 90–97 m)
  - 16-231: 6 × 6 × 6 cube (216 colors): 16 + 36 × r + 6 × g + b (0 ≤ r, g, b ≤ 5)
  - 232-255: grayscale from dark to light in 24 steps

```bash
echo "\e[38;5;200m some_text"
```

4. 24-bit true color 也有自己的独特的格式

- `<ESC>[38;2;r;g;bm`, r, g, b 对应色的 RGB

```bash
echo "\e[38;2;100;1;100m some_text"
```

## OSC(Operating System Command)

1. OSC 大部分由 xterm 定义, Xterm 选择可以用 BEL (0x07) 或者标准 ST (0x9C or 0x1B 0x5C) 来结束命令, 这部分很多时候由终端决定

   - OSC = `<ESC>]` ...some bytes... `<ST>` or `<BEL>`

2. Fs Escape, OSC 接一个 0x60-0x7E 之间的为 ISO-IR registry 可以用来控制输入

3. Fp Escape, 0x30-0x3F, 一些 private-use, 比如记住光标位置, 回溯光标

4. `<OSC>52;c;text\a`, 是可以控制系统剪切版的 Sequence, 此处的 text 需要时 base64 编码, c 是指定的剪贴板, 可以是 c(clipboard)、p(primary)、s(Secondary) 等，分别代表不同的剪切板, p, s 不太常用

```bash
# 设置剪切板为 hello
# 以 OSC 开头, 设置剪切板为 c, 接 base64 文本, 接终止符号 \a
echo -ne "\e]52;c;$(echo hello | base64)\a"
```

5. `<OSC>0;text\a`, 是可以控制终端标题

6. 其他还有很多, 可以问 chatgpt, 基本就是终端控制
