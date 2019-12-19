# Python

[TOC]

## 简介

![龟叔](http://www.yiyuan-tv.club/upload/2019/12/18861873-c637dde0488023f0-a291443999a046e9a01edd17581b0fd1.png)

> Python是著名的“龟叔”Guido van Rossum在1989年圣诞节期间，为了打发无聊的圣诞节而编写的一个编程语言。

### 发展史

- 1989年，Guido van Rossum为打发时间，决定为当时正构思的一个新的脚本语言编写一个解释器。作为python的狂热粉丝，他以Python命名该项目，使用C进行开发。
- ……
- 　2016年，Python 3.6
-　2018年，Python 3.7

![排行榜](http://www.yiyuan-tv.club/upload/2019/12/ranking_list2019-c010aeb366bf4d29ac033b957fc37942.png)

### 优缺点

#### 优点

- Python程序简单易懂，初学者入门容易。
- 开发效率高，有强大的第三方库，可以在基础库的基础上再开发，降低开发周期。
- 优雅，明确，简单
- ……

#### 缺点

- 运行速度慢，和C程序相比非常慢（对于500ms的网络延迟来说，501ms和510ms并没有什么区别)
- 代码不能加密



## 安装python

### MacOs

- [官网下载地址](https://www.python.org/downloads/mac-osx/)
- brew安装

```shell
brew install python
```



## 第一个程序 

### 控制台

```python
➜  ~ python3.7
Python 3.7.4 (default, Jul  9 2019, 18:15:00)
[Clang 10.0.0 (clang-1000.11.45.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> print('hello world')
hello world
>>>
```

### 文件



```python
#!/usr/local/bin/python3

print('hello world')
```

### 输入输出

> 所有输入为str类型,需要num类型要转

```python
>>> name = input('please input your name:\n')
please input your name:
yukang
>>> print(name)
yukang
```



## python的数据类型

### 整数

例如：`1`，`100`，`-8080`，`0` `0xa5b4c3d2`等等。

### 浮点数

如`1.23`，`3.14`，`-9.01`，等等

1.23x109就是`1.23e9`，或者`12.3e8`，0.000012可以写成`1.2e-5`

### 布尔值

直接用`True`、`False`表示布尔值（请注意大小写），也可以通过布尔运算计算出来

```python
print(3 > 5)

foo = None
bar = 'bar'

if foo:
    print("foo")
if bar:
    print('bar')

```

```
False
bar
```

### 字符串

> 输出 `I'm "ok"!`        `r`加载字符串之前表示不转义,对字符串来说`'`和`"`效果是一样的

```python
print('I\'m OK!')
print(r"I'm OK!")
```

> 输出
>
> I am learning
>
> python

```python
print('i am learning \npython')
print('''i am learning 
python''')
```

> 输入name并输出为 i am {name}       `f`加在前面可以是识别{}内的变量

```python
name = input('please input your name:\n')
print('i am ' + name)
print('i am', name)
print('i am {}'.format(name))
print(f'i am {name}')
```

