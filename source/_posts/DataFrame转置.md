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
print(df)
# >>
#    A  B  
# 0  1  5
# 1  2  6
# 2  3  7
# 3  4  8
df = df.T
print(df)
# >>
#    0  1  2  3
# A  1  2  3  4
# B  5  6  7  8
df = df.transpose()
```
这两个函数灵活性都不高，只能全列全行转置。
## unstack 方法
Dataframe中如果有多重索引，该方法可用于收缩索引数据，使其变成新的一列，例如
```python
print(s)
# >>
# one  a   1.0
#      b   2.0
# two  a   3.0
#      b   4.0
# dtype: float64
# 
s.unstack(level=-1,fill_value=None)
print(s)
#      a   b
# one  1.0  2.0
# two  3.0  4.0
```
参数level默认为-1，即默认收缩最后一个索引层级，fill_value指填充空值，默认为None
所以，实现指定行的转置为列，思路可以是先将需要转置的值转化为索引，再通过unstack方法，使其变成新的一列，以下是一个例子；
在最开始，df的索引是默认的，重新将id设置为df的索引后，只剩下A和B两列，unstack 将指定层级（至少两级索引，否则索引名为空值，交换后的新列名也是空）的索引名与所有列名交换，使指定索引名变为新列名
```python
df = pd.DataFrame({"A": ['x','y','z','o'],"B": [5,6,7,8],"id":['001','003','004','002']})
print(df.index)
# >>RangeIndex(start=0, stop=4, step=1)
df.set_index(["id"],inplace=True)
print(df)
# >>
#      A  B
# id       
# 001  x  5
# 003  y  6
# 004  z  7
# 002  o  8
df = df.unstack(level=-1)
print(df)
# >>
#    id 
# A  001    x
#    003    y
#    004    z
#    002    o
# B  001    5
#    003    6
#    004    7
#    002    8
# dtype: object
df = df.reset_index()
df.columns = ["class","id","values"]
print(df)
# >>
#    class  id values
# 0     A  001      x
# 1     A  003      y
# 2     A  004      z
# 3     A  002      o
# 4     B  001      5
# 5     B  003      6
# 6     B  004      7
# 7     B  002      8
```
## pivot方法


