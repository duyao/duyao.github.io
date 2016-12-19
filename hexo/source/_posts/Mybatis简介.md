title: Mybatis简介
date: 2015-10-10 09:45:41
tags: Mybatis
categories: note
---

Mybatis主要是使用配置文件来完成操作

## config.xml

这里可以配置数据库的基本信息

## SqlSession
* 作用
    - 向sql语句传入参数
    - 执行sql语句
    - 获取sql语句的结果
    - 事物的控制
* 如何获得SqlSession
    - 通过配置文件获取数据库相关信息
    - 通过信息构建SqlSessonFactory
    - 通过SqlSessionFactory打开数据库会话

<!--more-->

* 查询的配置文件，用来对应，一个JavaBean对应一个配置文件，对应数据库中的一张表

```xml
//namespace相当于包名，是一个作用域，可以保证id不重复
<mapper namespace="User">
//resultMap的type是查询时候的类型javaBean，id随意，与下面的id重复也没有关系
  <resultMap type="UserAlias" id="UserResult">
    //resultMap下面包含2中类型，一个是id一个是result,两者内容是相同的
    //数据库中是主键，用id，其余用result
    //column是数据库中的名字，jdbcType中java.sql.Type中的名字，property是javaBean中的名字
    <id column="id" jdbcType="INTEGER" property="id"/>
    <result column="username" jdbcType="VARCHAR" property="username"/>
    <result column="password" jdbcType="VARCHAR" property="password.encrypted"/>
    <result column="administrator" jdbcType="BOOLEAN" property="administrator"/>
  </resultMap>

//每个select的id都不相同，parameterType是参数的类型，resultMap对应上面的id
  <select id="queryMessageList" parameterType="long" resultMap="UserResult">
    SELECT * FROM user WHERE id = #{id:INTEGER}
  </select>

  <select id="version" parameterType="long" resultType="int">
    SELECT version FROM user WHERE id = #{id,jdbcType=INTEGER}
  </select>

  <delete id="delete" parameterType="UserAlias">
    DELETE FROM user WHERE id = #{id:INTEGER}
  </delete>

  <insert id="insert" parameterType="UserAlias" useGeneratedKeys="false">
    INSERT INTO user
    ( id,
    username,
    password,
    administrator
    )
    VALUES
    ( #{id},
    #{username,jdbcType=VARCHAR},
    #{password.encrypted:VARCHAR},
    #{administrator,jdbcType=BOOLEAN}
    )
  </insert>

  <update id="update" parameterType="UserAlias">
    UPDATE user SET
    username = #{username,jdbcType=VARCHAR},
    password = #{password.encrypted,jdbcType=VARCHAR},
    administrator = #{administrator,jdbcType=BOOLEAN}
    WHERE
    id = #{id,jdbcType=INTEGER}
  </update>

  <!--   Unique constraint check -->
  <select id="isUniqueUsername" parameterType="map" resultType="boolean">
    SELECT (count(*) = 0)
    FROM user
    WHERE ((#{userId,jdbcType=BIGINT} IS NOT NULL AND id != #{userId,jdbcType=BIGINT}) OR #{userId,jdbcType=BIGINT} IS
    NULL)  <!-- other than me -->
    AND (username = #{username,jdbcType=VARCHAR})
  </select>
</mapper>
```












