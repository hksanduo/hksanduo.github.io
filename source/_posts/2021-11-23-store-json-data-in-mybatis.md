---
title: "mybatis中typeHandler自定义实现typeHandler与数据库映射JSON读取"
date:   2021-11-23  12:07:00 +0800
layout: post
tag:
- Java
categories:
- Code
---

mybatis中typeHandler自定义实现typeHandler与数据库映射JSON读取

------
## 背景
mysql从5.7.版本开始支持json列，但是本质上仍然存储的是一个字符串，比起直接用varchar来说，它有专门对于json的的检索方式和修改方法。
在jdbc规范中，还没json类型的定义。所以对象一般都是用String属性，映射数据库的json列。在存储和读取的时候，需要自己完成json的序列化和反序列化。
在使用MyBatis的框架，可以通过定义TypeHandler来自动完成Json属性的序列化和反序列化。

## 环境
mybatis-spring-boot-starter:1.3.2
mysql:5.7.35
org.json:20210307

## 自定义 typeHandler 实现json从数据库中读取
将 mysql 中 json 格式的数据与 mybais 转化
### 步骤
#### 1、继承BaseTypeHandler类
BaseTypeHandler有四个方法，分别是：
```
 @Override
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, Object o, JdbcType jdbcType) throws SQLException {

    }

    @Override
    public Object getNullableResult(ResultSet resultSet, String s) throws SQLException {
        return null;
    }

    @Override
    public Object getNullableResult(ResultSet resultSet, int i) throws SQLException {
        return null;
    }

    @Override
    public Object getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        return null;
    }
```
我这里重写了JsonArrayHandler和JsonObjectHandler两个对象。

JsonArrayHandler.java示例：     
```
package cn.blingsec.engine.utils;

import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;
import org.json.JSONArray;
import org.json.JSONException;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class JsonArrayHandler extends BaseTypeHandler<JSONArray> {

    public JSONArray delResult(String jsonSource) throws SQLException {
        if (jsonSource != null) {
            JSONArray jsonArray;
            try {
                jsonArray = new JSONArray(jsonSource);
            } catch (JSONException ex) {
                throw new SQLException("There is an error converting JSONArray to json format for the content:" + jsonSource);
            }
            return jsonArray;
        }
        return null;
    }

    @Override
    public void setNonNullParameter(PreparedStatement ps,
                                    int i,
                                    JSONArray parameter, //需要转换的类型,JSON类型
                                    JdbcType jdbcType) throws SQLException {
        ps.setString(i, parameter.toString());
    }

    @Override
    public JSONArray getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return delResult(rs.getString(columnName));
    }

    @Override
    public JSONArray getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return delResult(rs.getString(columnIndex));
    }

    @Override
    public JSONArray getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return delResult(cs.getString(columnIndex));
    }
}
```

JsonObjectHandler.java 示例：   
```
package cn.blingsec.engine.utils;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class JsonTypeHandler<T extends Object> extends BaseTypeHandler<T> {
    private static final ObjectMapper mapper = new ObjectMapper();
    private Class<T> clazz;
    public JsonTypeHandler(Class<T> clazz) {
        if (clazz == null) throw new IllegalArgumentException("Type argument cannot be null");
        this.clazz = clazz;
    }

    @Override
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, T t, JdbcType jdbcType) throws SQLException {
        preparedStatement.setString(i, this.toJson(t));
    }
    @Override
    public T getNullableResult(ResultSet resultSet, String s) throws SQLException {
        return this.toObject(resultSet.getString(s), clazz);
    }

    @Override
    public T getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return this.toObject(rs.getString(columnIndex), clazz);
    }

    @Override
    public T getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        return this.toObject(callableStatement.getString(i), clazz);
    }

    private String toJson(T object) {
        try {
            return mapper.writeValueAsString(object);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    private T toObject(String content, Class<?> clazz) {
        if (content != null && !content.isEmpty()) {
            try {
                return (T) mapper.readValue(content, clazz);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        } else {
            return null;
        }
    }
}
```
#### 2、配置
重写TypeHandler这四个方法,在配置文件中需要自行配置，我在application.properties中配置，也可以在spring-mybatis.xml (数据源配置的文件)中配置。    
application.properties相关配置如下：
```
mybatis.type-handlers-package=cn.blingsec.engine.utils.JsonArrayHandler,cn.blingsec.engine.utils.JsonObjectHandler
```

> spring-mybatis.xml配置可以参考以下内容，来源：https://www.cnblogs.com/han-guang-xue/p/12832074.html
> ```
> <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
>    <!-- 数据源配置 -->
>    <property name="dataSource" ref="dataSource" />
>    <!-- 自动扫描mapping.xml文件 -->
>    <property name="mapperLocations" value="classpath:mapper/*/*.xml"></property>
>    <!-- 加入自定义typeHandler -->
>    <property name="typeHandlers">
>    <!-- 示例加入多个 typeHandler -->
>        <list>
>            <bean class="com.xingshu.utils.typehandler.JSONArrayHandler"></bean>
>            <bean class="com.xingshu.utils.typehandler.JSONObjectHandler"></bean>
>        </list>
>    </property>
></bean>
> ```

#### 3、mapper xml文件中配置
```
<insert id="updateComponentAnalysisCache">
    INSERT INTO
    component_analysis_cache(cache_type, last_occurrence, result, target,target_host,target_type,uuid)
    VALUES
    (#{ComponentAnalysisCache.cacheType},#{ComponentAnalysisCache.lastOccurrence},
    #{ComponentAnalysisCache.result,typeHandler=cn.blingsec.engine.utils.JsonObjectHandler},#{ComponentAnalysisCache.target},
    #{ComponentAnalysisCache.targetHost},#{ComponentAnalysisCache.targetType},
    #{ComponentAnalysisCache.uuid})
</insert>
```
我这里需要存储json数据的参数是：result

> 参考资料中有这样一个描述：
> 属性名后面 typeHandler 可以直接去掉, Mybatis 会自动映射,mybatis对jdbctype 和 typeHandler 会有一个优先级的匹配,如果实现类只有一个,可以省略不写
> 但是我在测试的时候会提示找不到jdbctype，报错信息，java方向实属太弱，没时间调试，留个尾巴，哪位大佬如果清楚可以沟通沟通：
> ```
> Analyzing error : org.mybatis.spring.MyBatisSystemException: nested exception is org.apache.ibatis.type.TypeException: Could not set parameters for mapping: ParameterMapping{property='ComponentAnalysisCache.result', mode=IN, javaType=class java.lang.Object, jdbcType=null, numericScale=null, resultMapId='null', jdbcTypeName='null', expression='null'}. Cause: org.apache.ibatis.type.TypeException: Error setting non null for parameter #3 with JdbcType null . Try setting a different JdbcType for this parameter or a different configuration property. Cause: org.apache.ibatis.type.TypeException: Error setting non null for parameter #3 with JdbcType null . Try setting a different JdbcType for this parameter or a different configuration property. Cause: java.sql.SQLException: Invalid argument value: java.io.NotSerializableException

>```
运行结果：
![20211123-01.png](/images/20211123-01.png)

## 参考
- [https://www.cnblogs.com/han-guang-xue/p/12832074.html](https://www.cnblogs.com/han-guang-xue/p/12832074.html)【mybatis中typeHandler自定义实现json的读写 】
- [https://juejin.cn/post/6861845363889078286](https://juejin.cn/post/6861845363889078286)【MyBatis通过TypeHandler自动编解码对象的Json属性】
- [https://segmentfault.com/a/1190000038259923](https://segmentfault.com/a/1190000038259923)【springboot mybatis 配置】