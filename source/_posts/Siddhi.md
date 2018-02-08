---
title: Siddhi
date: 2018-01-17 20:40:00
tags:
---
# Siddhi
Siddhi是一个复杂事件流程引擎CEP(Complex Event Processing)。使用类SQL的语言描述事件流任务，可以很好的支撑开发一个可扩展的，可配置的流式任务执行引擎。性能管理系统之中，告警模块采用storm作为告警生成组件。传统设计之中，为了支持不同的告警规则类型，我们需要编写不同的业务逻辑代码，但是使用了Siddhi之后，我们只需要配置不同的流任务Siddhiql，即可以支持不同的告警业务。

## Siddhi 能做什么？

- 简单 ETL:使用类SQL
- 基于 window 聚合：基于时间窗口
- 多个流 Join
- Pattern Query：`Pattern allows event streams to be correlated over time and detect event patterns based on the order of event arrival.`比如：在一天内，出现一次取现金额 < 100之后，同一张卡，再次出现取现金额 > 10000，则认为是诈骗。
- Sequence Query:和 pattern 的区别是，pattern 的多个 event 之间可以是不连续的，但 sequence 的 events 之间必须是连续的。我们可以看个例子，用sequence 来发现股票价格的 peak：

```
from every e1=FilteredStockStream[price>20], 
           e2=FilteredStockStream[((e2[last].price is null) and price>=e1.price) or ((not (e2[last].price is null)) and price>=e2[last].price)],
           e3=FilteredStockStream[price<e2[last].price] 
select e1.price as priceInitial, e2[last].price as pricePeak, e3.price as priceAfterPeak 
insert into PeakStream ;
```
上面的查询的意思， 
e1，收到一条 event.price>20。
e2，后续收到的所有 events 的 price，都大于前一条 event。
e3，最终收到一条 event 的 price，小于前一条 event。
ok，我们发现了一个peak。

## 集成到 JStorm
我将 Siddhi core 封装成一个 Siddhi Bolt，这样可以在 JStorm 的 topology 中很灵活的，选择是否什么方案，可以部分统计用 brain，部分用 Siddhi，非常简单。

## 参考：
- [Siddhi初探(内含一个实例)](https://www.cnblogs.com/coshaho/p/7051126.html)
- [CEP简介](https://www.cnblogs.com/XzcBlog/p/4468679.html)
- [让Storm插上CEP的翅膀 - Siddhi调研和集成](http://doc.okbase.net/fxjwind/archive/195904.html)

