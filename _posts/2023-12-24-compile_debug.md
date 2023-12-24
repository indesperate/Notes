# Compile and Debug flow

## Compile

一般来讲，编译的过程分为四个阶段：预处理、编译、汇编、链接

.c -> .i -> .s -> .o -> .out

cc1, cc1plus, as, collect2/ld

- 使用 strace 追踪 gcc 编译过程

use strace to trace the gcc compile process

```bash
# -f trace child process
# -qq quiet mode
strace -f -qq gcc test.c 2>&1 | nvim -
```

- 使用 gdb 追 踪 gcc 编译过程

gdb-internals.txt

```
set debuginfod enabled on
set detach-on-fork off
set follow-fork-mode child
set breakpoint pending on

file /usr/bin/gcc

break execve
commands
    silent
    echo \n\n\n--------------execve-----------------\n
    printf "execve: %s\n", $rdi
    set $argv = (char **) $rsi
    set $i = 0
    while ($argv[$i])
        printf "argv[%d] %s \n", $i, $argv[$i]
        set $i = $i + 1
    end
    echo \n\n\n-------------------------------------\n
end

run test.c
```

gdb -x gdb-internals.txt

## Debug

gdb 是如何实现的呢

- 如何在程序任意处设置断点, 使用中断

- gdb 修改.text 文件, 将断点处指令变为 int \$3

```asm
int $3
```

- 由于 x86 有单字节指令, 但是中断一般为 2 字节, 但是由于 int \$3 常用于 debug, 所以其其实为 1 字节, 0xcc

- windows 下 debug 下内存填充 0xcc 的原因也是这个, 用于追踪 segmentation fault

- windows 下 windbg 使用 WriteProcessMemory()

- linux 下使用 ptrace, observer and control another exection of a process, examine and change the tracee's memory and registor

- mprotect set protection on memory
