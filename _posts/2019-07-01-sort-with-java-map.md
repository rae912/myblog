---
layout: post
title: "使用Java Map按key排序和按Value排序"
subtitle: "Sort with java Map"
author: "qingshan"
header-img: "img/home-bg-o.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
---

今天工作中发现居然可以使用Java来优雅的对数据进行排序。核心技术用的是Java TreeMap的比较器：`Comparator`。此方法的优点是无需写冗长的业务逻辑，直接在初始化Map的时候指定，即可获得排序后的结果。可以根据需要，设置按Key排序，还是按Value排序。

#### 0. 理论基础
>* Map是键值对的集合接口，它的实现类主要包括：HashMap,TreeMap,Hashtable以及LinkedHashMap等。
>* TreeMap：基于红黑树（Red-Black tree）的 NavigableMap 实现，该映射根据其键的自然顺序进行排序，或者根据创建映射时提供的Comparator 进行排序，具体取决于使用的构造方法。
>* HashMap的值是没有顺序的，它是按照key的HashCode来实现的，对于这个无序的HashMap我们要怎么来实现排序呢？参照TreeMap的value排序。
>* Map.Entry返回Collections视图。

#### 1. 按Key排序
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
```

输出：
```shell
d:aaaaa
c:bbbbb
b:ccccc
a:ddddd
```

#### 2. 按Value排序
根据TreeMap的value来进行排序。对value排序我们就需要借助于Collections的sort(List<T> list, Comparator<? super T> c)方法，该方法根据指定比较器产生的顺序对指定列表进行排序。但是有一个前提条件，那就是所有的元素都必须能够根据所提供的比较器来进行比较，如下：

```java
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
```

输出：
```shell
d:aaaaa
c:bbbbb
b:ccccc
a:ddddd
```

#### 3. 升序或降序
如果需要改变默认升序或者降序，只需要改变`o1.getValue().compareTo(o2.getValue())`中o1和o2的顺序即可。

