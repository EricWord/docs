[TOC]

# 1.grouping sets/grouping_id/cube/rollup

这几个分析函数通常用于OLAP中，不能累加，而且需要根据不同维度上钻和下钻的指标统计，比如分小时、天、月的UV数

## 1.1 grouping sets

在一个group by 查询中，根据不同的维度组合进行聚合，等价于将不同维度的group by 结果进行union all



## 1.2 cube

根据group by的维度的所有组合进行聚合



## 1.3 rollup

是cube的子集，以最左侧的维度为主，从该维度进行层级聚合

##  