title: Mybatis 标签
date: 2015-10-14 20:49:25
tags: Mybatis
categories: note
---


# Mybatis 标签
## where 标签
用于处理select语句and等问题
```xml
<select id="version" parameterType="long" resultType="int">
	SELECT column1, column2, columnN FROM table_name;
	<where>
	  	<if test="condition">
	  		 column1=#{value1}
	  	</if>
  	</where>
  </select>

```

## set标签
用于处理update逗号，逗号等问题，与where类似
```xml
<update id="update" parameterType="UserAlias">
    UPDATE table_name 
    <set>
        <if test="condition">
	        column1 = #{value1}, column2 = #{value2}
	  	</if>
    </set>
</update>
```

##sql标签
sql标签与select等标签同级，相当于常量定义
```xml
<sql id="select_content">column1, column2, columnN</sql>

<select id="version" parameterType="long" resultType="int">
	SELECT <include refid="select_content"></include> FROM table_name;
</select>
```
<!--more-->
##trim标签
灵活的代替set或者where
```xml
<trim prefix="set" suffixOverrides=","></trim>
<trim prefix="where" prefixOverrides="and"></trim>
```

##choose标签
可以当做`if`...`else`，也可以视为`switch`
```xml
<choose>
	 <when test=""></when>
	 <when test=""></when>
	 <when test=""></when>
	 <otherwise></otherwise>
 </choose>
```

##collection与association标签
collection用来展示联合查询时子表内容的
association用来展示联合查询主表内容的
```xml
<collection property="javaBean的主表名" resultMap="Namespace.ResultMap的id"/>

<association property="javaBean的子表名" resultMap="Namespace.ResultMap的id></association>
```

##标签总结
![all lable][1]


  [1]: http://7xilc8.com1.z0.glb.clouddn.com/Mybaits%E6%A0%87%E7%AD%BE.png