---
title: mybatis-generator使用方式及实例
date: 2018-02-11 20:09:26
tags: mybatis
---

# mybatis-generator使用方式及实例
MyBatis Generator (MBG) 是一个Mybatis的代码生成器，它可以帮助我们根据数据库中表的设计生成对应的实体类，xml Mapper文件，接口以及帮助类，也就是我们可以借助该类来进行简单的CRUD操作。这样就避免了我们每使用到一张表的数据就需要手动去创建对应的类和xml文件，这就帮我们节约了大量的时间去开发和业务逻辑有关的功能。下面我主要介绍基于Maven和普通的Java工程两种方式来生成相应的文件。

> 注：如果对联合查询和存储过程您仍然需要手写SQL和对象。

## MySQL环境准备
- 安装：brew install mysql
- 启动：mysql.server start
- 运行：mysql_secure_installation
- 新建库manager--表tasks

```sql
CREATE TABLE IF NOT EXISTS tasks (
  task_id INT(11) NOT NULL AUTO_INCREMENT,
  subject VARCHAR(45) DEFAULT NULL,
  start_date DATE DEFAULT NULL,
  end_date DATE DEFAULT NULL,
  description VARCHAR(200) DEFAULT NULL,
  PRIMARY KEY (task_id)
) ENGINE=InnoDB;
```
## 配置
mybatis-generator有三种用法：命令行、eclipse插件、maven插件。本文使用maven插件方式，原因：该方式可以直接集成到项目中，从而方便后续其他人迭代新增表后，自动生成。

参考文档中有详细说明，如果不想细看，可以直接clone代码 https://github.com/DanielJyc/mybatis-generator 。然后：

- 更改`generatorConfig.xml`中，jar路径：`/Users/daniel/gitworkspace/JavaEETest/Test27-mybatis2/lib/mysql-connector-java-5.1.40.jar`为自己的真实路径。
- 更改表配置信息：

```xml
<table tableName="tasks" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false"
       enableSelectByExample="false" selectByExampleQueryId="false">
```

- 更改数据库连接信息

```
db.driver=com.mysql.jdbc.Driver
db.url= jdbc:mysql://localhost:3306/manager
db.username=root
db.password=****
```

- 根目录`generator`下，直接执行`mvn mybatis-generator:generate`即可。

## 集成到项目中
可以集成到项目中，两种方式：

- 作为单独的mvn模块。
- 直接集成到dal层。

新建子mvn模块后，把代码拷贝过去即可。

# 参考文档
- [sql教程](https://www.yiibai.com/mysql/create-table.html)
- [Mac下记录使用Homebrew安装Mysql全过程](http://blog.csdn.net/lkxlaz/article/details/54580735)
- [ Mybatis Generator最完整配置详解](http://blog.csdn.net/testcs_dn/article/details/77881776)
- [利用mybatis-generator自动生成代码](https://www.cnblogs.com/yjmyzz/p/4210554.html)
- [idea + mybatis generator + maven 插件使用](https://www.cnblogs.com/Erma-king/p/6694516.html)




