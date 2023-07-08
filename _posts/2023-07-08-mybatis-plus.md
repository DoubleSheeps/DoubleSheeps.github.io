---
layout:       post
title:        "MyBatis-Plus学习笔记"
date:       2023-07-08 13:53:10
author:       "Young Sheep"
header-style:      text 
catalog:      true
tags:
    - MyBatis-Plus
    - SpringBoot
    - Java开发笔记
---
>MyBatis是一个很简易的连接数据库的框架，只需要在xml中编写相应的SQL语句就可以实现数据库的增删改查操作，而且通过其Generator插件，可以直接根据数据库字段自动生成xml、DAO层以及DAO层对象，已经十分的无脑操作了，但其自动生成的Mapper仅包含最基本的增删改查方法，业务逻辑稍微复杂一点，就需要手撸xml中的sql语句了。  
>
>于是乎MyBatis-Plus出现了，它简直是懒癌患者的福音：
>* 直接省去了DAO层对象，在实体层的定义的对象通过注解即可与数据库中的表对应
>* 同时内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
>* 通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错

### 引入依赖
引入MyBatis-Plus插件就不用再引入MyBatis插件了，一方面可能会有冲突，另一方面MyBatis-Plus是在MyBatis基础上的增强插件，因此MaBatis-Plus对MyBatis原有的功能没有作任何删改，即使项目之前用的是MyBatis开发，现在换成MyBatis-Plus也不会有任何影响。
```java
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-boot-starter</artifactId>
	<version>3.3.1.tmp</version>
</dependency>
```
### 配置文件
```java
mybatis-plus:
    mapper-locations: classpath*:/mapping/*.xml
    #实体扫描，多个package用逗号或者分号分隔
    type-aliases-package: com.example.shirodemo.module.*.dataobject
    #type-handlers-package: com.github.niefy.common.handler
    global-config:
        #数据库相关配置
        db-config:
            #主键类型  AUTO:"数据库ID自增", INPUT:"用户输入ID", ID_WORKER:"全局唯一ID (数字类型唯一ID)", UUID:"全局唯一ID UUID";
            id-type: AUTO
			
            logic-delete-value: 0
            logic-not-delete-value: 1
        banner: false
    #原生配置
    configuration:
        map-underscore-to-camel-case: true
        cache-enabled: true
        call-setters-on-nulls: true
        jdbc-type-for-null: 'null'