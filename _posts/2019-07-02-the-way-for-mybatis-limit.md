---
layout: post
title: "MyBatis中Limit的几种方法"
subtitle: "The way for Mybatis limit"
author: "qingshan"
header-img: "img/home-bg-o.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
---

今天使用Spring Boot对数据库做CRUD的时候，需要对结果进行排序和limit。因为ORM使用的默认Mybaits，因此查找了下资料，总结一下：
#### 0. 最符合直观的原生MySQL
```java
Condition cond = new Condition(OpsLog.class);
cond.setOrderByClause ("id limit" + start+"," +20) // 直接拼接生成原生MySQL
```

#### 1. 大众用法PageHelper
```java
Condition cond = new Condition(OpsLog.class);
cond.createCriteria().andCondition("component =", "ETLAgent");
cond.setOrderByClause("create_time desc");
PageHelper.startPage(1, 20,false); // 每次查询20条

List<OpsLog> OpsList = opsLogService.findByCondition(cond);
```

#### 2. 小众用法RowBounds
```java

 public List<RepayPlan> listRepayPlan(int start) {
        // 查询所有未还款结清且应还日期小于当前时间的账单
        Example example = new Example(RepayPlan.class);
        example.orderBy("id "); // 按id排序
        example.createCriteria()
                .andNotEqualTo("repayStatus", 3)
                .andLessThanOrEqualTo("shouldRepayDate", new Date());
        RowBounds rowBounds = new RowBounds(start, 20); //　每次查询20条
        return epaymentPlanMapper.selectByExampleAndRowBounds(example,rowBounds);
}
```

#### 3. 结论
推荐用 RowBounds：mybatis自带的，且速度快。明显比PageHelper快。

#### 参考资料
[资料链接](https://blog.csdn.net/jiangyu1013/article/details/90140532)