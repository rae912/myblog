---
layout: post
title: "JPA报错No property found for type的问题"
subtitle: "Solve the exception for JPA No property found for type problem"
author: "qingshan"
header-img: "img/post-sample-image.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
---

在 JPA 中定义完 model 和 dao 之后，JPA 会自动检索 model，然后生成 findByFieldName 的方法。

例如，定义了一个字段叫 actionid，JPA 在 repository 文件里，会自动创建 findByActionid 的方法，意思是根据 actionid 这个字段来查询，获得结果。

但是，如果字段是带下划线&quot;_&quot;的时候，生成的方法是 findByAction_id，结果 100% 会报错，提示：

```bash
No property action found for type Action!
```

这是因为 JPA 虽然会生成 findByAction_id，但是 Spring 却不会认下划线，最终是按照下划线前的关键字，也就是`action`来解析，model 和数据库里没有这个字段，当然会报错了。

经过实践，有两种方式解决这个问题：

# 1. @Column 别名指定

在创建 model 的时候，可以使用@Column 指定数据库实际的字段，那么定义 model 字段名的时候，就可以设置任意别名了。例如：

```java
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "Action_ID")
    private Integer actionid;
```

定义的字段是 actionid，没有下划线，但是实际上映射到数据库是 Action_ID 这个字段是有下划线的。但是这个过程已经是在查询的最后阶段，不会影响程序运行，完美解决问题。

# 2. @Query 指定 SQL

如果方法 1 不能满足业务需求，可以使用@Query 在 repository 中来指定方法所执行的 SQL 语句，覆盖原来自动方法。例如：

```java
public interface TableMigratePolicyRepository extends JpaRepository<TableMigratePolicy, String> {
    @Query("SELECT t FROM TableMigratePolicy t WHERE t.action_id = 0 or t.comment is not null or t.comment <> '' ")
    List<TableMigratePolicy> findByAction_id(Integer action_id);
}
```

这时候，即使方法名里有下划线，但是实际运行的是`@Query`里执行的 SQL 语句，也会避免报错的问题。