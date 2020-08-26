---
layout:     post
title:      数据库
author:     Liber Sun
header-img: img/post-basic.jpg
catalog: true
tags:
    - hibernate
    - mybatis
---

> hibernate与mybatis都是一种在应用层与数据存储层进行交互的技术。

# 数据库操作的演化

为了更清除的认知彼此，我们需要首先了解Java操作数据库的发展与变革。

## jdbc

JDBC(JAVA DataBase Connectivity), 通过JDBC我们可以在java程序和数据库之间建立链接，并通过java程序来操作数据库中的数据。

JDBC将应用程序开发者与底层的数据库驱动程序进行了解耦，是一个承上启下的中间层。

JDBC(是一系列的接口)不能直接访问数据库，必须依赖数据库厂商提供的JDBC驱动程序。JDBC为用户完成了以下三步工作：

01. 同数据库建立链接，获取Statement。
02. 向数据库发送SQL语句，JDBC API使用SQL语句来完成创建（create）、读取（read）、更新（update）和删除（delete）（CRUD）操作，由Statement执行
03. 处理从数据库返回的数据ResultSet

这类代码还在很大程度上依赖于SQL，而SQL并非是跨数据库的标准，这使得从一种数据库移植到另一种数据库变得困难起来。同时程序员需要处理太多的细节，一个简单的查询，就需要一大推代码，打开Connection、创建Statement，执行SQL，遍历ResultSet，获取结果，最后还必须记得关闭Connection。**另外我们通过JDBC执行SQL语言操作数据库这与JAVA面向对象的概念又是相违背的。**

## ORM

关系数据库是一张二维表格的集合体，利用JDBC的操作方式是面向表格的，不是面向对象的，使用JDBC的整个过程面向对象编程的思维观念很大程度上被遏制了。

鉴于这个问题，在使用Java开发时，**我们希望真正的建立一个对象型数据库，或者说至少使用起来看起来像一个对象型数据库**，但是，目前常用的数据库又的确是关系型数据库，这一点短期内又无法改变。

`ORM由此出现` ，ORM是JDBC的一层封装，提供了以面向对象的思维来操作数据库中的表的方式，ORM工具框架最大的核心就是封装了JDBC的交互，你不在需要处理结果集中的字段或者行或者列。借助于ORM可以快速进行开发，而无需关注JDBC交互细节。

ORM(对象关系映射)，使得 `对象` 和 `数据库中的表` 一一对应，构建了一个直接的映射关系。

ORM的出现是和面向对象编程的大潮息息相关的，它透明地把应用对象持久到关系数据库中的表的技术，它提供功能来执行完整的CRUD操作并鼓励面向对象的查询。

随着各种ORM框架的发展，最终形成了两个流派，及Hibernate和Mybatis。

### Hibernate

Hibernate直接**把Java对象映射到数据库表上**，把java对象的属性用声明式的方式映射到数据库表中。Hibernate提供了一个完整的ORM解决方案，但不提供对查询的控制权，对于不太熟悉SQL的面向对象编程者来说，Hibernate是最佳选择。

### MyBatis

MyBatis不提供完整的ORM解决方案，也不提供任何的对象和关系模型的直接映射，他是一个半自动化的持久层框架。MyBatis给你提供了对查询的全面控制权，他以SQL为中心，要求全面地控制SQL, **将java对象映射到SQL查询的结果**上。

## JPA

JPA(Java Persistence API)意即Java持久化API，是Sun官方在JDK5.0后提出的Java持久化规范（JSR 338，这些接口所在包为javax.persistence）
JPA的出现主要是为了**简化持久层开发**以及**整合ORM**技术，结束Hibernate、TopLink、JDO等ORM框架各自为营的局面。JPA是在吸收现有ORM框架的基础上发展而来，易于使用，伸缩性强。
总的来说，JPA包括以下3方面的技术：

* **ORM映射元数据**： 支持XML和注解两种元数据的形式，元数据描述对象和表之间的映射关系
* **API**： 操作实体对象来执行CRUD操作
* **查询语言**： 通过面向对象而非面向数据库的查询语言（JPQL）查询数据，避免程序的SQL语句紧密耦合

JPA(Java persistence API)，用于持久化的API，能够使应用程序以统一的方式访问持久层。

那么如何实现JPA呢？还是得依靠ORM思想。基于ORM思想实现的、满足JPA规范的框架就是JPA框架。

### SpringDataJPA

Spring Data JPA是Spring Data家族的一部分，可以轻松实现基于JPA的存储库。 
此模块处理对基于JPA的数据访问层的增强支持。 它使构建使用数据访问技术的Spring驱动应用程序变得更加容易。其在JPA的基础上添加了另一层的抽象(Repository层)，极大地简化了持久层开发以及ORM框架切换的成本。

![Spring Data JPA和JPA的关系](https://upload-images.jianshu.io/upload_images/10458268-de1860c2f3941031.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/623/format/webp)

spring推出的ORM框架，在jpa中是不存在dao层的, 它被 `repository` 替换了， `repository` 对dao层的代码进行了封装。

在利用Spring boot data JPA进行表设计的时候，表对象之间经常存在各种映射关系，如何正确将理解的映射关系转化为代码中的映射关系是关键之处。

> 同时，SpringJPA为NOSQL数据库同样提供了一套OGM的策略，

# hibernate案例

在使用hibernate时，我们需要定义一个java类，然后借用hibernate自动在数据库中为我们构建一个表，对类的操作就是对数据库中的表的操作，当然如果你用了hibernate，那么表这个概念应该不在存在，而以一切皆对象的方式进行业务代码的开发。

``` java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Entity
@Table (name = "m_user")
public class User{
    @Id
    @GeneratedValue
    private Integer id;

    @Column (nullable = false, unique = true,length=30)
    private String username;

    @Column (nullable = false)
    private String password;
  
    @CreatedDate
    private Long createTime;

    @LastModifiedDate
    private Long updateTime;
}
```

上述代码通过若干注解实现了 类与数据库表的一一对应关系。

# mybatis案例

而对于mybatis而言，在于对sql查询回来的result与定于的类进行映射。

``` java
public class TileSQLProvider {
    public String getShpInfo(@Param (value = "name") String name) {
        String sql = "select  \n" +
                "a.st_xmax,a.st_ymax,a.st_xmin,a.st_ymin,b.type,c.props \n" +
                "from \n" +
                "(select ST_XMax(extend),ST_YMax(extend),ST_XMin(extend),ST_YMin(extend) from (select ST_Extent(geom) as extend from " + name + ") as foo) as a,\n" +
                "(SELECT ST_GeometryType(geom)  as type from " + name + " limit 1) as b,\n" +
                "(select string_agg(name,',') as props from (select COLUMN_NAME as name from information_schema.COLUMNS where table_name = '" + name + "') as p ) as c";
        return sql;
    }
}

@Mapper
public interface TileMapper {
    @Results ({
            @Result (column = "st_xmax", property = "xMax", jdbcType = JdbcType.DOUBLE),
            @Result (column = "st_ymax", property = "yMax", jdbcType = JdbcType.DOUBLE),
            @Result (column = "st_xmin", property = "xMin", jdbcType = JdbcType.DOUBLE),
            @Result (column = "st_ymin", property = "yMin", jdbcType = JdbcType.DOUBLE),
            @Result (column = "type", property = "type", jdbcType = JdbcType.VARCHAR),
            @Result (column = "props", property = "props", jdbcType = JdbcType.VARCHAR)
    })
    @SelectProvider (type = TileSQLProvider.class, method = "getShpInfo")
    ShpInfo getShpInfo(@Param (value = "name") String name);
}
```
@Results的column与property建立了表与类ShpInfo的一一对应关系。


# hibernate和mybatis的比较

mybatis和hibernate都是ORM的实现，前者注重于SQL，在使用mybatis时，我们往往是在深思熟虑之后，在已经构建好数据库表的情况下再进行开发。后者注重于业务模型，在使用hibernate时，没有数据库表的概念，因此我们可以任意的进行业务模型（数据库存储的实体）的更改。

同时由于关注于对象，hibernate是与数据库无关的，其移植性较强，而mybatis会在sql中出现了数据库特有的函数。

对于常规的业务开发，我更钟情于Hibernate，其提供的基础模板能让我们快速的进行开发与迭代。而在针对Gis系统研发时，我们将面临大量的空间数据操作，为了使用postgresql中的postgis模块，这时就需要使用mybatis，利用sql语句调用postgis的相关函数。

