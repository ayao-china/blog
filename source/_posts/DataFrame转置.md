---
title: DataFrame转置
date: 2022-07-12 21:21:51
tags:
categories: [python,pandas]
top_img: http://tva1.sinaimg.cn/large/006UraCCly1h3zw9tub3mj32yo1o0nph.jpg
cover: http://tva1.sinaimg.cn/large/006UraCCly1h3zw9tub3mj32yo1o0nph.jpg
---
# DataFrame转置
## 前言：
Dataframe转置有其内置的函数，可以用其他较为灵活的方法实现，以下总结了三种转置的方法，对应不同的需求场景，参考文档：https://www.cnblogs.com/traditional/p/11967360.html，https://blog.csdn.net/qq_18351157/article/details/105931547
## 内置函数
df.T 和 df.transpose() 都是是pandas内置的转置方法，但是区别在于transpose() 可以有更多参数可以调节，例如：df.transpose(copy=True),但是df.T必须重新赋值
```python
df = pd.DataFrame({"A": [1,2,3,4], "B": [5,6,7,8]})
df = df.T
print(df)
# >>
# 0  1  2  3
# A  1  2  3  4
# B  5  6  7  8
df = df.transpose()
```
这两个函数灵活性都不高，只能全列全行转置。
## unstack 方法

