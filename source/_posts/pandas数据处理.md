---
title: pandas基础知识总结
date: 2022-07-07 21:05:40
sticky: true
categories: 
- [python,pandas]
top_img: http://tva1.sinaimg.cn/large/006UraCCly1h3zwalsfmyj31hc0wpn6j.jpg
cover: http://tva1.sinaimg.cn/large/006UraCCly1h3zwalsfmyj31hc0wpn6j.jpg
---

# pandas基础知识总结
## 前言
Pandas 是 Python 语言的一个扩展程序库，用于数据分析，广泛应用在学术、金融、统计学等各个数据分析领域。
Pandas 的主要数据结构是 Series （一维数据）与 DataFrame（二维数据）
基础教程参考：https://www.runoob.com/pandas/pandas-tutorial.html
以下为个人在工作中总结的扩展知识点

## merge 函数
参考文档：https://blog.csdn.net/brucewong0516/article/details/82707492
```python

    def merge(
        left,
        right,
        how: str = "inner",
        on=None,
        left_on=None,
        right_on=None,
        left_index: bool = False,
        right_index: bool = False,
        sort: bool = False,
        suffixes=("_x", "_y"),
        copy: bool = True,
        indicator: bool = False,
        validate=None,
    )
```
1. left_index: 如果为True，则使用左侧DataFrame中的索引（行标签）作为其连接键。 对于具有MultiIndex（分层）的DataFrame，级别数必须与右侧DataFrame中的连接键数相匹配。
  right_index与left_index功能相似。
2. 以index为链接键,需要同时设置left_index= True和right_index=True，或者left_index设置的同时，right_on指定某个Key。总的来说就是需要指定left、right链接的键，可以同时是key、index或者混合使用。

## DataFrame 遍历
参考文档：https://blog.csdn.net/qq_27575895/article/details/90034037
1. iterrow(),是实际应用中使用最频繁的遍历方式，它可以将行索引和每行数据（series）一起遍历，但是无法在遍历中修改对象的值
```python

for ind,row in df.iterrows():
    print(ind,row,row['key_name'])
```
ind 返回本行索引，row返回本行数据（类型为series），如果需要定位到单个元素，row['key_name']即可

2. 指定行或列的遍历可使用lambda 函数，例如：
```python

df['key_name'] = df['key_name'].apply(lambda x: int(x))
```

