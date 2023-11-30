---
title: "Combinator"
categories:
  - Blog
tags:
  - Python
  - Functional programming
last_modified_at: 2023-11-30
---

# Y Combinator

## 如何实现匿名函数的递归

- 比如如下的递归阶乘函数
```python
def fact(n):
    if n == 1:
        return 1
    else:
        return n * fact(n - 1)
```

- 当我们使用lambda时, 无法获得fact函数名, 无法实现递归

```python
fact = lambda n : 1 if n == 1 else n * ?(n - 1)
```

- 一种直观的想法就是传函数进去, 函数参数f作为函数名的替代一直传递

```python
def recur(f, n):
    if n == 1:
        return 1
    else:
        return n * f(f, n - 1)
recur(recur, 10);
```

- 我们把n提取出来

```python
def recur(f):
    return lambda n : 1 if n == 1 else n * f(f)(n - 1)
recur(recur)(10)
```

- 如此我们把上面的recur(recur)也写为lambda

```python
fact = (lambda f: f(f))(lambda f: lambda n : 1 if n == 1 else n * f(f)(n - 1))
```

- 如此便实现了匿名函数的递归, 但是我们想让他更通用一点应该如何? 我们把通用部分f(f)先提取出来, 让他变为lambda一个参数, 如此便只剩下函数部分

```python
fact = (lambda f: f(f))(lambda f: lambda n : (lambda ff :1 if n == 1 else n * ff(n - 1))(lambda n : f(f)(n)))
```

- 把函数部分提取出来, 调换下位置, 把n放在最内部, 以此, 外部调用时n为最外围输入

```python
(lambda n : (lambda ff: 1 if n == 1 else n * ff(n - 1))(lambda n : f(f)(n)))
(lambda ff : (lambda n: 1 if n == 1 else n * ff(n - 1)))(lambda n : f(f)(n))
```

```python
Y = lambda func: (lambda f: f(f))(lambda f: func(lambda n : f(f)(n)))

fact = Y(lambda f : (lambda n : 1 if n == 1 else n * f(n - 1)))
fact(10)

```

- 这其实就是函数式编程的"Y Combinator", 最后的表达式非常漂亮, 匿名f(n - 1)直接调用

# S combinator

```python
S = lambda f : lambda g: lambda x: f(x)(g(x))
```

