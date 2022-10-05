---
title: Python基础
date: 2022-07-07 20:54:00
categories:
- [PYTHON]
top_img: http://tva1.sinaimg.cn/large/006UraCCly1h3zwb6kpvvj31z40pd7qz.jpg
cover: http://tva1.sinaimg.cn/large/006UraCCly1h3zwb6kpvvj31z40pd7qz.jpg
---

# Python基础

## 前言

Python 是一种解释型、面向对象、动态数据类型的高级程序设计语言。相比于其他的编程语言，入门较为简单，对于那些想要学习编程的非计算机专业的同学非常友好。  
本篇主要总结日常使用python 时遇到的问题和一些扩展，

## 环境搭建与基础语法

环境搭建：参考 https://www.runoob.com/python/python-install.html  
阿里云镜像：http://mirrors.aliyun.com/

1. Python中默认的编码格式是 ASCII 格式，在没修改编码格式时无法正确打印汉字，所以在读取中文时会报错，解决方法：在文件开头加入以下任意一行代码即可

```python
   
# -\*- coding: UTF-8 -\*-
# coding=utf-8
```
   Pycharm编码设置：

   - 进入 **file > Settings**，在输入框搜索 **encoding**。
   - 找到 **Editor > File encodings**，将 **IDE Encoding** 和 **Project Encoding** 设置为utf-8

## 数据类型
### 字符串 str
1. 参考链接：https://www.runoob.com/python3/python3-string.html
2. 扩展：

### 列表 list
1. 参考链接：https://www.runoob.com/python/python-lists.html
2. 扩展： 在实际应用中，列表推导式的使用概率较为频繁，例如：
```python
   
ls = [1,2,3,4]
ls2 = [x + 1 for x in ls]
```
还可以使用 enumerate()函数遍历列表，这样可同时遍历list的下标索引和元素，例如：
```python

for ind,enu in enumerate(ls):
    print(ind,enu)
```
### 元组 tuple
1. 参考链接： https://www.runoob.com/python3/python3-tuple.html
2. 扩展：元组经常与列表结合使用，例如：
    ```
   
   ls = [(3,5),(6,7),(9,10)]
   print([x[1] for x in ls])
   >>[3,6,9]
   ```
   元组是不可变的，重新赋值的元组 tup，绑定到新的对象了，而不是修改了原来的对象
### 字典 dict
1. 参考链接： https://www.runoob.com/python3/python3-dictionary.html
2. 扩展：访问字典的所有键：dict.keys()
        访问字典的所有值：dict.values()
        但以上两种方式返回的是一个视图对象，想要继续使用，需要转成列表或元组类型，例如：list(dict.keys()) or tuple(dict.values()) 
3. 普通字典转为有序字典（按插入顺序输出） collections.OrderedDict()
```python

import collections
dict2 = collections.OrderedDict()
dict2['a'] = 'A'
dict2['b'] = 'B'
dict2['c'] = 'C'
>>OrderedDict([('a', 'A'), ('b', 'B'), ('c', 'C')])
```
### 集合 set
1. 参考链接：https://www.runoob.com/python3/python3-set.html
2. 扩展：集合与列表的区别在于，集合是一个无序的不重复元素序列,而列表是有序的可以重复的序列，允许使用索引访问，所以当需要对一个list 去重时，可以先转成set，再转回list,例如：
```python

ls1 = [1,2,3,4,4,4,5,6]
ls2 = list(set(ls1))
print(ls2)
# >> [1,2,3,4,5,6]
```
## 循环与条件控制
### 循环 for/while
1. 基本用法： https://www.runoob.com/python3/python3-loop.html
2. 扩展：
   break 语句可以跳出 for 和 while 的循环体。如果你从 for 或 while 循环中终止，任何对应的循环 else 块将不执行；
   continue 语句被用来告诉 Python 跳过当前循环块中的剩余语句，然后继续进行下一轮循环；
   pass是空语句，是为了保持程序结构的完整性。 pass 不做任何事情，一般用做占位语句

### 条件控制 if else
1. 基本用法： https://www.runoob.com/python3/python3-conditional-statements.html
2. 扩展：
   break 和 continue无法在if中使用，但可以在循环嵌套if的情况下使用 例如：
```python
   
for x in [1,2,3,4,5]:
  if x> 3:
      break
  else:
      continue

```
## 面向对象
### 类
1. 基本用法： http://c.biancheng.net/view/2283.html
2. 装饰器：以下为定义装饰器函数及使用装饰器的方法，在调用funB过程中，会执行操作一和操作二
```python

def funA(fn):
    # ...操作一
    fn()
    #...操作二
    return 
@funA
def funB():
    ...
```
   

