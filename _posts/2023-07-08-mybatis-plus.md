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
    #如果在 Mapper 中有自定义方法(XML 中有自定义实现)，需要配置，告诉 Mapper 所对应的 XML 文件位置
    mapper-locations: classpath*:/mapping/*.xml
    #别名包扫描路径，通过该属性可以给包中的类注册别名，注册后在 Mapper 对应的 XML 文件中可以直接使用类名，而不用使用全限定的类名(即 XML 中调用的时候不用包含包名)
    type-aliases-package: com.example.shirodemo.modules.sys.dataobject
    global-config:
        #数据库相关全局配置
        db-config:
            #主键默认类型  AUTO:"数据库ID自增", INPUT:"用户输入ID", ID_WORKER:"全局唯一ID (数字类型唯一ID)", UUID:"全局唯一ID UUID";
            id-type: AUTO
            #逻辑删除
            logic-delete-field: status #表示删除的字段 默认deleted
            logic-delete-value: 0   #表示删除的值
            logic-not-delete-value: 1   #表示未删除的值
        #取消控制台输出的logo
        banner: false
    #原生配置
    configuration:
        #指定当结果集中值为 null 的时候是否调用映射对象的 Setter（Map 对象时为 put）方法
        #配置为true后，即使列为空，那么依然会调用列的Setter方法，这时候我们在Setter方法可以做对null的处理
        call-setters-on-nulls: true
```
### 实体层注解实现与数据库映射
常用注解：
* @TableName("表名") 设置映射的表名
* @TableId(type = IdType.AUTO) 指定主键以及主键的生成方式，不用此注解默认主键名为id字段
* @TableField(xxx) 可以指定字段的一些属性
	    "字段名"名字不是直接驼峰命名转化的时候需要定义字段在表中的名称
		insertStrategy = FieldStrategy.IGNORED 字段插入时重复则忽略插入
		exist=false 解决实体类存在的属性对应的表中没有，不设置则会报错
		
```java
@Data
@TableName("user_info")
public class UserModel {
    @TableId(type = IdType.AUTO)
    private Integer id;

    private String name;

    private String password;

    private Byte status;

    @TableField(insertStrategy = FieldStrategy.IGNORED)//account重复则不插入
    private String account;

}
```
### Mapper继承通用Mapper类实现的增删改查
```java
@Mapper
public interface UserMapper extends BaseMapper<UserModel> {
    //也可以自定义方法，然后编写xml文件配置sql语句实现
    List<UserModel> selectTeacher();

}
```
### Service层继承通用IService类，实现基础的Service逻辑
```java
public interface UserService extends IService<UserDO> {

}
```
实现类也要继承通用实现类，配置实体类和Mapper
```java
public class UserServiceImpl extends ServiceImpl<UserDOMapper, UserDO> implements UserService {

}
```
基础的Service增删改查
```java
//查询所有
userService .list(lambdaQuery);
//查询数量
userService .count(lambdaQuery);
//根据ID集合查list集合
userService .listByIds(ids);
//根据ID删除
userService .removeById(id);
//根据ID集合删除
userService .removeByIds(ids);
//修改
userService .update(userModel);
//新增
userService .save(userModel);
//新增或者修改,判断ID是否存在，如果ID不存在执行新增，如果ID存在先执行查询语句，查询结果为空新增，否则修改。
userService.saveOrUpdate(userModel)
//批量新增或者修改
userService.saveOrUpdateBatch(userModelList)
```
