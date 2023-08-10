---
layout:       post
title:        "Mybatis学习笔记"
date:       2023-08-10 17:41:04
author:       "Young Sheep"
header-style:      text
catalog:      true
tags:
    - Java开发笔记
    - SpringBoot
    - Mybatis
    - MyBatis-Plus
---
>之前项目采用了mybatis-plus插件作为数据库的连接层，大大提高了数据库基础增删改查的效率，但遇到复杂的数据库查询操作LambdaQueryWrapper我还是玩不太明白，因此只能尝试老办法：编写对应mapper的xml实现。好在mybatis-plus只是在mybatis的基础上做拓展，原先mybatis支持的xml方式映射mapper方法依然是可用的。

### if字段的使用
在编写xml中的select方法时，可以使用\<if\>\</if\>中的test属性根据参数是否存在动态的构造查询语句。
例如当参数中name属性存在时就要根据name字段进行查询学生数据库时，可这样编写xml中的select方法：
```sql
<select id="xxxx" parameterType="xxxx" resultType="xxxx">
	select student_info.* from student_info
	<if test="name != null and name != ''">
	where student_info.name = #{name,,jdbcType=VARCHAR}
	</if>
</select>
```
显然这种简单的情况用LambdaQueryWrapper也能轻松实现，但LambdaQueryWrapper处理多表联查就没那么方便了，但编写xml还是一样的逻辑。
例如现在的需求还要加上如果参数中的classId属性存在时就要从student_classs表中找class_id相同的学生信息，这同样可以根据编写带有\<if\>\</if\>的xml实现：
```sql
<select id="xxxx" parameterType="xxxx" resultType="xxxx">
	select student_info.* from student_info
	<if test="classId != null">
	, student_class
	</if>
	where true
	<if test="name != null and name != ''">
	and student_info.name = #{name,,jdbcType=VARCHAR}
	</if>
	<if test="classId != null">
	and student_class.class_id = #{classId,jdbcType=INTEGER}
	and student_class.student_id = student_info.id
	</if>
</select>
```
需要注意的是由于我们两个查询条件都是通过\<if\>\</if\>判断的，因此为了更简洁的编写，我先固定了where true的查询条件，这个条件并不会对最终的结果产生影响，但这样做两个需要通过if判断的查询条件就可以直接用and并列了，不用考虑第一个查询条件如果不存在就没有where字段了。

### foreach字段的使用
实际中更常见的需求可能是上述的class_id不止一个，也就是班级id具有多选的功能，因此得到的classIds是一个数组，而在select查询中使用in条件判断就能满足这样的需求，也就是可以用class_id in (x,x,x)实现，但如何在xml中构造这样的查询条件就很关键了。
\<foreach\>\</foreach\>简直就是为构造这个查询语句而生的，在xml中使用\<foreach\>字段，我们可以设置collection属性规定要循环的数组，item属性规定循环的每个值，open规定前缀，separator规定循环的分割符，close规定后缀，因此上述需求的select方法就可这样编写：
```sql
<select id="xxxx" parameterType="xxxx" resultType="xxxx">
	select student_info.* from student_info
	<if test="classIds != null and classIds.length > 0">
	, student_class
	</if>
	where true
	<if test="name != null and name != ''">
	and student_info.name = #{name,,jdbcType=VARCHAR}
	</if>
	<if test="classIds != null and classIds.length > 0">
	and student_class.class_id in
	<foreach collection="classIds" item="classId" open="(" separator="," close=")">
	  #{classId}
	</foreach>
	and student_class.student_id = student_info.id
	</if>
	group by student_info.id
</select>
```
需要注意的是由于从多个班级中查询学生信息，因此可能存在重复的信息，可通过group by进行去重。同时数组我们在用if判断其数量大于0时我们要用length属性，但如果foreach的是List或Set时我们就要用size属性。