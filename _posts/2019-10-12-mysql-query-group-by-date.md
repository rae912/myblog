---
layout: post
title: "Mysql根据日期来分组统计数据"
subtitle: "Group by and Analysis with date in Mysql"
author: "qingshan"
header-img: "img/post-bg-brain.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
---

今天在工作中，需要用到MySQL做根据日期的聚合查询，带一点统计计算性质。经过检索，发现MySQL本身就支持简单的计算统计功能，例如，按日期分组统计的查询可以这样写：

``` SQL
SELECT DATE_FORMAT( deteline, "%Y-%m-%d %H" ) , COUNT( * ) 
FROM test
GROUP BY DATE_FORMAT( deteline, "%Y-%m-%d %H" ) 
```

备注：
查询某天：deteline, "%Y-%m-%d"
某时：deteline, "%Y-%m-%d %H"

原理其实就是对dateline进行处理，然后再对处理后的数据分组



