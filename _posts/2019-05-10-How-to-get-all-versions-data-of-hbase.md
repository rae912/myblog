---
layout: post
title: "Java Hbase API 获取不同版本的数据"
subtitle: "How to get all versions data of hbase with Java API"
author: "qingshan"
header-img: "img/post-bg-e2e-ux.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
---


最近工作用需要使用Java Hbase API来获取Hbase数据，使用最新的3.0版本。网上的资料大多数都是1.x和2.0的，许多的方法已经过时了，最新的写法应该是下面这样：
```java
    private List<Map<String, String>> findDataWithRowKey(String tName, String rowKey, String[] dateRange) {

        TableName tableName = TableName.valueOf(tName);
        List<Map<String, String>>  ReturnResult = new LinkedList<>();
        Get get = new Get(rowKey.getBytes()); //根据主键查询
//        Integer maxVersion = get.getMaxVersions(); // 获取最大版本数量

        try {
            Connection con = ConnectionFactory.createConnection(config);
            Table table = con.getTable(tableName);

            Result r = table.get(get);

            for (Cell cell : r.rawCells()) {
                //时间戳转换成日期格式
                String timestampFormat = new SimpleDateFormat("yyyy-MM-dd HH:MM:ss").format(new Date(cell.getTimestamp()));

                logger.info("\nKeyValue: " + cell.toString());
                logger.info("key: " + new String(cell.getRowArray(), "utf-8"));

                Map<String, String> HbaseData = new HashMap<>();
                HbaseData.put("rowKey", rowKey);
                HbaseData.put("family", new String(cell.getFamilyArray(), "utf-8"));
                HbaseData.put("qualifer", new String(cell.getQualifierArray(), "utf-8"));
                HbaseData.put("value", new String(cell.getValueArray(), "utf-8"));
                HbaseData.put("timestamp", timestampFormat);
                ReturnResult.add(HbaseData);

            }
        } catch (Exception e) {
            e.printStackTrace();
            logger.error(e.toString());
        }
        return ReturnResult;
    }
```

但是上述的代码默认只能获取一个数据版本的数据，因此，如果需要获取所有版本的数据，应该加上这一句：
```java
get.readAllVersions();  // 读取所有版本
```

根据文档，还可以根据时间戳获取指定时间范围的版本号，因为Hbase的版本就是以时间戳标识的：
```java
get.setTimeRange(startTime, endTime); // 经过实践，这个逻辑无效，原因未知
```
但是经过时间，这一句代码实现的效果，只能获取到endTime这一天的数据，而startTime并没有生效，原因未知。
进一步发现使用
```java
get.getTimeRange()
```
获得的数据，就是long型数据结构所能表达的最大值和最小值，猜测可能是有bug存在。
```shell
maxStamp=9223372036854775807, minStamp=0
```


因此，可以使用readAllversions()的逻辑，再结合时间戳过滤数据，达到变相获取指定时间范围版本数据的目的。

关键代码如下：
```java
if (dateRange.length != 0) { // 没有日期范围时，取所有版本
    startTime = (new SimpleDateFormat("yyyy-MM-dd")).parse(dateRange[0], new ParsePosition(0)).getTime();
    endTime = (new SimpleDateFormat("yyyy-MM-dd")).parse(dateRange[1], new ParsePosition(0)).getTime() + 24 * 3600 * 1000; // 多取一天的值
    logger.info("指定时间戳查询数据：" + String.valueOf(startTime) + ", " + String.valueOf(endTime));

//                get.setTimeRange(startTime, endTime); // 经过实践，这个逻辑无效，原因未知
    get.readAllVersions();  // 读取所有版本，后面逻辑中过滤
}

// 中间省略从rawCells获取数据的逻辑


if (startTime != 0) { // 如果时间日期生效，则过滤掉不在这个日期中的数据
    if (cell.getTimestamp() < startTime || cell.getTimestamp() > endTime) {continue;}
}

```


至于为什么get.getTimeRange() 和 get.setTimeRange(startTime, endTime) 异常，还需要进一步读取源代码分析。
