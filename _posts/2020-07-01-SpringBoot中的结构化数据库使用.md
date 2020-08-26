---
layout:     post
title:      SpringBoot中的结构化数据库使用
author:     Liber Sun
header-img: img/post-basic.jpg
catalog: true
tags:

    - SpringBoot
    - postgresql
    - Spring Data JPA
    - MyBatis
    - 

---

>>> 虽然Spring Data JPA已经帮我们对数据的操作封装得很好了，约定大于配置思想，帮我们默认了很多东西。JPA（Java持久性API）是存储业务实体关联的实体来源。它显示了如何定义一个面向普通Java对象（POJO）作为一个实体，以及如何与管理关系实体提供一套标准。因此，javax.persistence下面的有些注解还是必须要去了解的，以便于更好地提高工作效率。

# 1. 基本注解

* @Entity，定义的对象将会成为被JPA管理的实体，将映射到指定的数据库表。
* @Table，指定数据库的表名。
* @Id，设定数据库的主键，不设置时会默认查找id字段
* @IdClass，
* @GeneratedValue，主键生成策略，有4种类型Table、SEQUENCE, IDENTITY, AUTO。
* @Transient，表示该属性并非一个到数据库表的字段的映射，表示非持久化属性。
* @Column，定义列名。
  + name
  + unique
  + nullable
  + length，仅类型为String时有效
  + precision，数值总长度
  + scale，小数个数
* @Tempora，定义Date类型映射到数据库对应字段的精度。
* @Enumerated，映射为枚举。
* @Lob，将属性映射成数据库支持的大对象类型，支持Clob（长字符串类型），Blob（字节类型）两种数据库类型的字段，其占用空间大，往往结合@Basic(fetch=FetchType. LAZY)将其设置为延迟加载。

  

# 2. 关联关系注解

* @JoinColumn，其主要配合@OneToOne、@ManyToOne、@OneToMany一起使用，单独使用没有意义，其体现了表与表之间的关联连接。
  + name 外键列名
  + referencedColumnName 外键引用的列名称，默认值为引用表的主键列
* @JoinTable，配合@ManyToMany使用，通过构建连接表，表达了多对多的关联连接。
  + name 连接表的名称，默认为两个关联的主实体表的连接名称，用下划线分隔。
  + joinColumns  连接表的外键列，它引用拥有关联的实体的主表。
  + inverseJoinColumns 连接表的外键列，它引用不拥有关联的实体的主表。
* @OneToOne 指定与具有一对一多重性的另一个实体的单值关联。在控制方通过 @JoinColumn(name="外键名", referencedColumnName="外键引用的表的主键列名")，在被控制方@OneToOne(mappedBy = "被控方的表名")。
* @OneToMany，表示了单对多的情况，其关联关系推荐不建立关系表，而是存储在多的一方，就是说多端为关系维护端，负责关系的增删改查。一端则为关系被维护端，不能维护关系。通过@JoinColumn(name = "外键名")指定外键名，不指定时会自动生成，而在单的一方定义@OneToMany(mappedBy = "主控方的对应的属性名")，表明是关系被维护端。
* @ManyToMany，多对多关系中一般不设置级联保存、级联删除、级联更新等操作，双方都需要@JoinTable指定连接表关系。如果其中关系的维护由控制端进行，控制端进行@JoinTable，另一端不进行操作，只用声明@ManyToMany(mappedBy="主控方的对应的属性名")。注意此时如果删除被控制端，首先需要解除关系。

在@ManyToMany，@ManyToMany；@OneToMany，@ManyToOne的中需要定义@JsonManagedReference指定主控，  @JsonBackReference指定被控，避免无限引用

# CascadeType

CascadeType. PERSIST 如果A类新增，对应的B类也会新增

CascadeType. MERGE  如果A类新增或更新，对应的B类也会新增或更新

CascadeType. REMOVE 如果A类删除, B类也会删除

CascadeType. REFRESH 获得A类时，会获取包含的最新的B类信息

CascadeType. DETACH 删除A类时，撤销所有相关的外键关联。

CascadeType. ALL 上述均有，强依赖于A类。

这里的A类是指控制方，B类是控制方

# @oneTomany() @ManyToOne()

``` java
    @OneToMany(cascade = CascadeType.ALL,mappedBy = "patrolRecord")
    @JsonManagedReference
    private List<Problem> problemList;
```

`@JsonManagedReference` 由少的一方 `patrolRecord` 来管理映射，以避免无限的递归获取Json。 `mappedBy` 指定需要管理的 `Problem` 的patrolRecord字段。

``` java
    @ManyToOne(cascade = CascadeType.REFRESH)
    @JsonBackReference
    @JoinColumn(name="patrol_record_id", referencedColumnName = "id")
    private PatrolRecord patrolRecord;
```

`@JoinColumn` 避免了额外的创建关联表，在 `Problem` 的表中使用特定的字段"patrol_record_id"进行关联。

我们在创建 `Problem` 和 `PatrolRecord` 时，首先创建 `PatrolRecord` , 然后再创建 `Problem` ，以"patrol_record_id"进行关联.

> Many的一方会存储其one的id；
> 而one的一方不会存储其Many的id。

# Hibernate的对象/关系映射

如何把数据库表的一个列和一个对象的一个字段进行映射，这就是Hibernate的映射所需要考虑的事情，这是通过Type来实现的。

## 基本值类型

基本值类型是指非聚合的java类型，即Java的基本数据类型，包括String、Character、Boolean、Byte、Short、Integer、Long、Float、Double、BigInteger、BigDecimal、java.util. Date、java.sql. Timestamp、 java.sql. Time、 java.sql. Date、java.util. Calendar、java.util. Currency、 java.util. Locale、java.util. TimeZone、java.net. URL、java.lang. Class、java.sql. Blob、java.sql. Clob、java.util. UUID、 java.io. Serializable

## 集合类型

即定义一个集合（注意这里与关联的差别），使用Hibernate的list type

``` java
@TypeDefs ({
        @TypeDef (name = "list-array", typeClass = ListArrayType.class),
        @TypeDef (name = "jsonb", typeClass = JsonBinaryType.class),
           @TypeDef(name = "json", typeClass = JsonStringType.class),
})
```

``` java
@Type(type = "list-array")
    @Column(
        name = "sensor_ids",
        columnDefinition = "uuid[]"
    )
    private List<UUID> sensorIds;
 
    @Type(type = "list-array")
    @Column(
        name = "sensor_names",
        columnDefinition = "text[]"
    )
    private List<String> sensorNames;
 
    @Type(type = "list-array")
    @Column(
        name = "sensor_values",
        columnDefinition = "integer[]"
    )
    private List<Integer> sensorValues;
 
    @Type(type = "list-array")
    @Column(
        name = "sensor_long_values",
        columnDefinition = "bigint[]"
    )
    private List<Long> sensorLongValues;
```

## 实体类型

> 在最开始的版本中，我们不得不去定义一个表，然后连接两个表来表示一个实体类型。

现在，我们可以采取两种策略，一种是自定义Type实现 org.hibernate.type. Type、 org.hibernate.usertype. UserType或者org.hibernate.usertype. CompositeUserType，其较为复杂。另一种是直接使用Hibernate封装好的json type。

``` java
@Type(type='jsonb')
@Column (columnDefinition = "jsonb")
private Location location;

@Type(type='jsonb')
@Column (columnDefinition = "jsonb")
private List<Location> location;


@Type(type='jsonb')
@Column (columnDefinition = "jsonb")
private Map<String,Location> location;
```

jsonb提供更强大的查询功能：

``` 
List<String> participants = entityManager.createNativeQuery(
    "SELECT jsonb_pretty(p.ticket) " +
    "FROM participant p " +
    "WHERE p.ticket ->> 'price' > '10'")
.getResultList();
```

daterange提供的查询功能：

``` 
List<Book> discountedBooks = entityManager
.createNativeQuery(
    "SELECT * " +
    "FROM book b " +
    "WHERE " +
    "   b.discount_date_range @> CAST(:today AS date) = true ", Book.class)
.setParameter(
    "today",
    LocalDate.of(2019, 12, 1)
)
.getResultList();
 
assertTrue(
    discountedBooks.stream().anyMatch(
        book -> book.getTitle().equals("High-Performance Java Persistence")
    )
);
```

## 枚举类型

@Enumerated(EnumType. STRING)
private MyEnums myenums;