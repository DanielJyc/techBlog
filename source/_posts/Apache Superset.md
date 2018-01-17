# Apache Superset
Superset其实是一个自助式数据分析工具，它的主要目标是简化我们的数据探索分析操作。

## 相似产品对比

| name | 描述 | 实现 |
| --- | --- | --- |
| grafana | grafana的druid插件，比较简陋，github一年不更新了<span class="Apple-tab-span" style="white-space:pre"></span> | js |
| Metabase<span class="Apple-tab-span" style="white-space:pre"></span> | 支持数据库种类多，启动方便，支持json查询。图形化查询，只能有一个聚合字段，两个维度<span class="Apple-tab-span" style="white-space:pre"></span> | Clojure<span class="Apple-tab-span" style="white-space:pre"></span> |
| imply-pivot<span class="Apple-tab-span" style="white-space:pre"></span> | 基于Plywood，部署方便，能构造复杂的查询。目前已经闭源了，没法二次开发<span class="Apple-tab-span" style="white-space:pre"></span> | Js |
| airbnb/superset<span class="Apple-tab-span" style="white-space:pre"></span> | 权限管理完善，图形可定制性也比较高，github持续更新,集合了metabase的Dashboard和pivot的查询可定制性优点，部署相对麻烦<span class="Apple-tab-span" style="white-space:pre"></span> | python+js<span class="Apple-tab-span" style="white-space:pre"></span> |

## 数据库支持
Superset 是基于 Druid.io 设计的，但是又支持横向到像 SQLAlchemy 这样的常见Python ORM框架上面。

Druid 是一个基于分布式的快速列式存储，也是一个为BI设计的开源数据存储查询工具。Druid提供了一种实时数据低延迟的插入、灵活的数据探索和快速数据聚合。现有的Druid已经可以支持扩展到TB级别的事件和PB级的数据了，Druid是BI应用的最佳搭档。

跟类似产品Hive相比，速度快了很多。

## 架构
整个项目的后端是基于Python的，用到了Flask、Pandas、SqlAlchemy。

**后端：**

- Flask AppBuilder(鉴权、CRUD、规则）
- Pandas（分析）
- SqlAlchemy（数据库ORM）

**前端：**

用到了npm、react、webpack,这意味着你可以在手机也可以流畅使用。

- d3 (数据可视化)
- nvd3.org(可重用图表)

## 局限性
- Superset的可视化，目前只支持每次可视化一张表，对于多表join的情况还无能为力
- 依赖于数据库的快速响应，如果数据库本身太慢Superset也没什么办法
- 语义层的封装还需要完善，因为druid原生只支持部分sql。



## 参考
- [解密Airbnb 自助BI神器：Superset 颠覆 Tableau](https://segmentfault.com/a/1190000005083953)
- [superset官网](http://airbnb.io/projects/superset/)
- [druid.io可视化调研](https://fangyeqing.github.io/2016/11/04/druid.io%E5%8F%AF%E8%A7%86%E5%8C%96%E8%B0%83%E7%A0%94/)
- [Grafana vs Superset](https://siftery.com/product-comparison/grafana-vs-superset)

