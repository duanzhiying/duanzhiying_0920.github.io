---
title: DbUtils配合HikariCP查询MySQL/Presto/Impala/Kylin数据源
date: 2017-03-14 17:31:35
tags: [DbUtils, HikariCP]
categories: 技术
toc: true
---

## 场景描述
大数据交互用户个性化定制报表，底层的数据源除了涉及到常规的MySQL数据库，还包括Presto、Impal、Kylin，将来还会有ElasticSearch。而这些对于操作的用户而言完全是透明的，他们感知到的只是有权限能查看到的数据表。
<!--more-->

底层的查询希望达到统一，Presto和Kylin均提供RESTful API接口，但MySQL显然并不支持，Impala还没有调研。但确定的是Presto、Impala和Kylin均对 ANSI-92 SQL语法支持。且这四种数据源均可以通过JDBC进行连接。考虑到DbUtils的轻量型和其他优点，以及提高性能的JDBC连接池HikariCP，计划搭配这两种来根据用户选择的数据表建立动态的JDBC链接并select到需要的数据展现给前端。
测试了DbUtils配合HikariCP查询MySQL/Presto/Impala/Kylin这四种数据源，均通过。期间遇到一些问题和BUG，故做如下记录。

## 具体测试
环境：IntelliJ IDEA，Java，Maven项目
使用工具：DbUtils轻量级JDBC工具类库，HikariCP JDBC连接池组件。maven依赖配置如下：
```xml
<!-- JDBC 连接池 -->
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP-java7</artifactId>
    <version>2.4.11</version>
</dependency>
<!-- JDBC 工具包 -->
<dependency>
    <groupId>commons-dbutils</groupId>
    <artifactId>commons-dbutils</artifactId>
    <version>1.6</version>
</dependency>
```

### 连接Mysql
四个数据源的连接测试方式相似，只是配置的信息不同。贴代码如下：
在main方法中，考虑到查询语句构建的动态性，因为不确定所以不可能每个数据表对应一个实体bean，所以测试用map list的方式返回；如果待操作的表是确定的，可以构建实体bean，如main方法汇总注释的部分，结果集类型为BeanListHandler处理。

**需要注意**：
如果是实体bean对应数据表，经常会出现数据表中字段名如app_id，而bean实体定义为appId这样的转换，解决方案有两种，一种是在构造语句时进行转换，一种是修改映射方法。官网解释如下：
>1. Alias the column names in the SQL so they match the Java names: select social_sec# as socialSecurityNumber from person
2. Subclass BeanProcessor and override the mapColumnsToProperties() method to strip out the offending characters.

``` Java
/**
 * Created by duanzhiying on 2017/3/13.
 */
public class MysqlTest {
    public static void main(String[] args) throws Exception{
        //首先根据数据库/表id获取该数据库的连接方式
        //得到后，构建对该数据库表的连接
        QueryRunner run = new QueryRunner(getDataSource());
        String sql = "SELECT id,app_id,app_owner,type,abstract_info FROM jdp_app_info";
        /*//如果是确定的实体bean，可以直接映射为bean的list
        ResultSetHandler<List<JdpAppInfo>> h = new BeanListHandler<JdpAppInfo>(JdpAppInfo.class);
        List<JdpAppInfo> appInfos = run.query(sql, h);*/
        //返回结果的map list
        List<Map<String, Object>> results = run.query(sql, new MapListHandler());
        Object o = JSONObject.toJSON(results);
        System.out.println(o);
    }

    /**
     * 获取数据源
     */
    public static DataSource getDataSource(){
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:mysql://192.168.200.124:3358/jdp?zeroDateTimeBehavior=convertToNull&" +
                "allowMultiQueries=true&noAccessToProcedureBodies=true");
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUsername("root");
        ds.setPassword("root");
        return ds;
    }
}
```

**执行结果**：
 ![MySQL测试结果](/images/posts/2017.3.14/mysql test.png)
**需要配置的Maven依赖**：
```xml
<!-- Mysql 连接 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>4.2.5.RELEASE</version>
</dependency>
```

### 连接Presto
测试连接的代码和MySQL相似，依然需要配置DataSource的jdbcUrl，driverClassName，userName和password。区别的是main方法：
```java
public static void main(String[] args) throws Exception{
        //这一句话一定要有，否则失败
        HttpUtil.isTest = true;
        //针对于presto，避免fillStatement方法，pmdKnownBroken设置为true
        QueryRunner run = new QueryRunner(getDataSource(),true);
        final String sql = "SELECT cast(1 as DOUBLE)/5 FROM test_server limit 1";
        List<Map<String, Object>> results = run.query(sql, new MapListHandler());
        System.out.println("list size: " + results.size());
    }
```

**需要注意**：
如果依然用测试Mysql的写法：new QueryRunner(ds)，会抛出如下异常：
`Exception in thread "main" java.sql.SQLException: Method PreparedStatement.getParameterMetaData is not yet implemented Query`
 ![Presto连接异常](/images/posts/2017.3.14/presto error.png)

 原因：
 PrestoPreparedStatement类对getParameterMetaData方法并没有实现，而是直接抛出异常。解决方案就是在构建QueryRunner时将参数pwdKnownBroken置为true，这样DbUtils的query方法执行时fillStatement方法不会执行调用getParameterMetaData方法，以避免抛出异常。
 ![PrestoPreparedStatement](/images/posts/2017.3.14/prestopreparedstatement.png)

**需要配置的Maven依赖**：
```xml
<!-- Presto 连接 -->
<dependency>
        <groupId>presto-jdbc-jd</groupId>
        <artifactId>presto-jdbc-jd</artifactId>
        <version>0.132</version>
</dependency>
```

### 连接Impala
Impala连接测试的方法和Presto相同，同样需要将pwdKnownBroken设置为true，否则会抛出SQLException，因为HivePreparedStatement类对getParameterMetaData方法也未实现，不支持且直接抛出异常。

**需要配置的Maven依赖**：
```xml
<!-- Impala 连接 -->
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-jdbc</artifactId>
    <version>2.1.0</version>
</dependency>
```

**需要注意**：
之前测试hive-jdbc的maven依赖版本号为0.13.0，抛出如下异常：
`Exception in thread "main" java.sql.SQLException: Method not supported`
截图如下：
 ![Impala连接异常](/images/posts/2017.3.14/impala_error.png)
原因是该版本的HiveConnector类并没有实现isReadOnly方法，会直接抛出异常。如下：
 ![HiveConnection方法不支持](/images/posts/2017.3.14/hive_connector.png)
在google搜索，该问题已经反馈过：`https://issues.apache.org/jira/browse/HIVE-11501`，目前高版本的hive-jdbc maven依赖已经解决该问题。version为2.1.0已经不会抛出该异常，可以正常连接。

### 连接Kylin
测试方法代码如下：
```java
public class KylinTest {
    public static void main(String[] args) throws Exception{
        //首先根据数据库/表id获取该数据库的连接方式
        //得到后，构建对该数据库表的连接
        QueryRunner run = new QueryRunner(getDataSource());
        final String sql = "SELECT\n" +
                "KYLIN_SALES.PART_DT\n" +
                ",KYLIN_SALES.LEAF_CATEG_ID\n" +
                ",KYLIN_SALES.LSTG_SITE_ID\n" +
                ",KYLIN_SALES.LSTG_FORMAT_NAME\n" +
                ",KYLIN_SALES.PRICE\n" +
                ",KYLIN_SALES.SELLER_ID\n" +
                "FROM KYLIN_SALES";
        List<Map<String, Object>> results = run.query(sql, new MapListHandler());
        System.out.println("list size: " + results.size());
    }

    /**
     * 获取数据源
     */
    public static DataSource getDataSource(){
        HikariDataSource ds = new HikariDataSource();
        //需要对HikariConfig的connectionTestQuery赋值
        ds.setConnectionTestQuery("select 1");
        ds.setJdbcUrl("jdbc:kylin://192.168.195.169:7070/learn_kylin");
        ds.setDriverClassName("org.apache.kylin.jdbc.Driver");
        ds.setUsername("ADMIN");
        ds.setPassword("KYLIN");
        return ds;
    }
}
```

**需要注意**：
如果在DataSource设置时没有对HiKariConfig的connectionTestQuery赋值，即注释掉`ds.setConnectionTestQuery("select 1");`这句代码，则会报如下异常：
`[main] ERROR com.zaxxer.hikari.pool.PoolBase - HikariPool-1 - Failed to execute isValid() for connection, configure connection test query. (null)`

![Kylin连接异常](/images/posts/2017.3.14/kylin_error.png)

报错原因：AvaticaConnection类对isValid方法并没有实现。因此通过设置HikariConfig类的connectionTestQuery字段，当该字段不为null时，HikariCP的PoolBase类的isUseJdbc4Validation属性会被赋为false，在这种情况下测试jdbc连接将通过connectionTestQuery字段设置的语句测试（如show tables），而不是去调用connection的isValid()方法。

**需要配置的Maven依赖**：
```xml
<!-- Kylin 连接 -->
<dependency>
    <groupId>org.apache.kylin</groupId>
    <artifactId>kylin-jdbc</artifactId>
    <version>1.6.0</version>
</dependency>
```

## 总结
DbUtils完美支持Mysql、Presto、Impala、Kylin数据源，下一步计划封装通用数据库连接查询方法。
