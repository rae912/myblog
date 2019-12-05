---
layout: post
title: "Java中根据字典的key和value排序"
subtitle: "Sort collection in Java"
author: "qingshan"
header-img: "img/post-bg-digital-native.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
---

今日在工作中遇到一个需求：
```
有一个数据类型为List<Map<String, Object>>，Key是指标名称，Value是该指标的值，可能是字符串，也可能是整型，也可能是浮点型。现在需要根据指定的指标名的值来进行排序，生成一个新的有序List。
```

# 1 根据value来排序
我是通过Java的Collection的Sort(List<T> list, Comparator<? super T> c)方法实现的，这个方法的示例如下：
```java
// 根据指定比较器产生的顺序对指定列表进行排序。但是有一个前提条件，那就是所有的元素都必须能够根据所提供的比较器来进行比较

import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;
import java.util.TreeMap;
public class TreeMapTest {
    public static void main(String[] args) {
        Map<String, String> map = new TreeMap<String, String>();
        map.put("a", "ddddd");
        map.put("c", "bbbbb");
        map.put("d", "aaaaa");
        map.put("b", "ccccc");
        
        //这里将map.entrySet()转换成list
        List<Map.Entry<String,String>> list = new ArrayList<Map.Entry<String,String>>(map.entrySet());
        //然后通过比较器来实现排序
        Collections.sort(list,new Comparator<Map.Entry<String,String>>() {
            //升序排序
            public int compare(Entry<String, String> o1,
                    Entry<String, String> o2) {
                return o1.getValue().compareTo(o2.getValue());
            }
            
        });
        
        for(Map.Entry<String,String> mapping:list){ 
               System.out.println(mapping.getKey()+":"+mapping.getValue()); 
          } 
    }
}

// 结果：
// d:aaaaa
// c:bbbbb
// b:ccccc
// a:ddddd
```

实际的业务需求，需要动态的指定排序的指标和顺序（或倒序），所以我是这样写的：
```java
    private List<Map.Entry<String, Integer>> orderBy(JSONArray array, String orderByMetric, String order) {
        Map<String, String> metricMap = new HashMap<>();
        metricMap.put("allocatedMB", "allocatedMB");
        metricMap.put("containers", "runningContainers");
        metricMap.put("progress", "progress");
        metricMap.put("duration", "duration");

        Map<String, Integer> map = new TreeMap<>();
        // 生成一个字典，key是json文本，value是allocatedMB，方便后面排序
        for (Object o: array) {
            JSONObject ob = JSONObject.parseObject(o.toString());
            map.put(ob.toString(), Integer.parseInt(ob.get(metricMap.get(orderByMetric)).toString()));
        }

        // 根据字典的value来排序并生成一个新的List
        List<Map.Entry<String, Integer>> list = new ArrayList<Map.Entry<String, Integer>>(map.entrySet());
        Collections.sort(list, new Comparator<Map.Entry<String, Integer>>() {
            @Override
            public int compare(Map.Entry<String, Integer> o1, Map.Entry<String, Integer> o2) {
                if (order.equals("asc")) {
                    return o1.getValue().compareTo(o2.getValue()); // 顺序
                } else {
                    return o2.getValue().compareTo(o1.getValue()); // 逆序
                }

            }
        });

        return list;
    }
```

至此已经完美的满足了业务需求，性能也还非常不错。那么如果下次要是要求用key来做排序依据呢？同样也是可以的。

# 2 通过Key来排序

TreeMap默认是升序的，如果我们需要改变排序方式，则需要使用比较器：Comparator。Comparator可以对集合对象或者数组进行排序的比较器接口，实现该接口的public compare(T o1,To2)方法即可实现排序，如下：

```java
import java.util.Comparator;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;
import java.util.TreeMap;
public class TreeMapTest {
    public static void main(String[] args) {
        Map<String, String> map = new TreeMap<String, String>(
                new Comparator<String>() {
                    public int compare(String obj1, String obj2) {
                        // 降序排序
                        return obj2.compareTo(obj1);
                    }
                });
        map.put("b", "ccccc");
        map.put("d", "aaaaa");
        map.put("c", "bbbbb");
        map.put("a", "ddddd");
        
        Set<String> keySet = map.keySet();
        Iterator<String> iter = keySet.iterator();
        while (iter.hasNext()) {
            String key = iter.next();
            System.out.println(key + ":" + map.get(key));
        }
    }
}

// 结果：
// d:aaaaa
// c:bbbbb
// b:ccccc
// a:ddddd
```

# 3 实现原理

* Map是键值对的集合接口，它的实现类主要包括：HashMap,TreeMap,Hashtable以及LinkedHashMap等。

* TreeMap：基于红黑树（Red-Black tree）的 NavigableMap 实现，该映射根据其键的自然顺序进行排序，或者根据创建映射时提供的 Comparator 进行排序，具体取决于使用的构造方法。
HashMap的值是没有顺序的，它是按照key的HashCode来实现的，对于这个无序的HashMap我们要怎么来实现排序呢？参照TreeMap的value排序。

* Map.Entry返回Collections视图。
