---
title: "VIM"
categories:
  - Blog
tags:
  - Vim
  - Terminal
last_modified_at: 2024-1-11
---

# VIM

Record vim tricks

## change fileformat

- encoding, 内部编码格式, 呈现的编码格式
- fileencoding, 文件的编码格式, 会被 vim 转成内部编码格式
- fileencodings, 逐一探测

```vim
set ff=unix
"set file save encoding
set fileencoding=cp936
```

```vim
"set file read encoding, edit file with cp936 fileencoding
e ++enc=cp936
```
