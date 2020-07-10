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

# CascadeType

CascadeType. PERSIST 如果A类新增，对应的B类也会新增

CascadeType. MERGE  如果A类新增或更新，对应的B类也会新增或更新

CascadeType. REMOVE 如果A类删除, B类也会删除

CascadeType. REFRESH 获得A类时，会获取包含的最新的B类信息

CascadeType.DETACH 删除A类时，撤销所有相关的外键关联。

CascadeType. ALL 上述均有

# @oneTomany() @ManyToOne()

``` java
    @OneToMany(cascade = CascadeType.ALL,mappedBy = "patrolRecord")
    @JsonManagedReference
    private List<Problem> problemList;
```

`@JsonManagedReference` 由少的一方`patrolRecord`来管理映射，以避免无限的递归获取Json。 `mappedBy` 指定需要管理的`Problem`的patrolRecord字段。

``` java
    @ManyToOne(cascade = CascadeType.REFRESH)
    @JsonBackReference
    @JoinColumn(name="patrol_record_id", referencedColumnName = "id")
    private PatrolRecord patrolRecord;
```

`@JoinColumn`避免了额外的创建关联表，在`Problem`的表中使用特定的字段"patrol_record_id"进行关联。


我们在创建`Problem`和`PatrolRecord`时，首先创建`PatrolRecord`,然后再创建`Problem`，以"patrol_record_id"进行关联.

>Many的一方会存储其one的id；
>而one的一方不会存储其Many的id。