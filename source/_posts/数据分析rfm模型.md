---
title: 数据分析-rfm模型
date: 2022-07-07 21:01:32
categories:
- [数据分析]
top_img: http://tva1.sinaimg.cn/large/006UraCCly1h4clzegh0mj31hc0u07wh.jpg
cover: http://tva1.sinaimg.cn/large/006UraCCly1h4clzegh0mj31hc0u07wh.jpg
---

# 数据分析-rfm模型
## 前言
 rfm模型是crm(用户关系管理系统)中经常使用的数据分析模型，R-最近一次消费至今天数(recency)，F-时间段内消费频次(frequency)，M-消费金额(monetray)  
 参考文档:  
 https://blog.csdn.net/m0_52000372/article/details/122511518  
 https://baijiahao.baidu.com/s?id=1724552575497144820&wfr=spider&for=pc
## 定义解释
### R-最近一次消费至今天数(recency)
计算用户最近一次消费至今的天数，一般可分为30天（一个月），90天（三个月），360（一年），720天（两年），这个指标与用户在此店铺的粘性有关，R值越小，说明用户最近对于目标店铺的记忆性越强，粘性也越大，同时还反映了用户的消费周期，便于预测用户的回购率。
### F-时间段内消费频次(frequency)
用户的消费频次是判断用户忠诚度的重要指标，用户的消费频次与用户忠诚度成正比，通过计算消费频次，可区分该时间段内的新老客和忠诚客户，而用户的分级对于运营策略的精准投放有重要作用。
### M-消费金额(monetray)
一定时间段内的M-消费金额(monetray)体现了用户的消费力度，M值越大，用户消费力度越大，对GMV的贡献度也越大，所以可以进一步通过运营策略留住这样高贡献度的用户，M值较小（贡献度较小）的用户，可以通过其他指标判断是否为已流失客户或者即将流失客户。
## 应用场景
结合rfm三个维度，将用户细分为以下等级：  
{% asset_img rfm.png 用户细分等级 %}



