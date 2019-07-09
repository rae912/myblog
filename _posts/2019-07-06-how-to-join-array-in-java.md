---
layout: post
title: "Java中join字符串列表"
subtitle: "How to join array in Java"
author: "qingshan"
header-img: "img/post-bg-rwd.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
---

在python等动态语言中，很容易就将一个字符串列表转换成字符串，例如：

Python中：
```python
a = ['a','b','c']
print(",".join(a))
```

JavaScript中：
```javascript
a = ['a','b','c']
a.join(",")
```

上面两种写法都根据内置的`join`关键字的分隔符输出 'a,b,c'。 但是找了一下，在Java中似乎并没有遇到类似join的关键字。经过搜索，发现了如下方法：

#### 1. StringBuilder 循环列表
```java
// Java program for convert character list to string 
import java.util.Arrays; 
import java.util.List; 
  
// Convert List of Characters to String in Java 
class GFG { 
    public static void main(String[] args) 
    { 
        // create character list and initialize 
        List<Character> str =  
                Arrays.asList('G', 'e', 'e', 'k', 's'); 
  
        // create object of StringBuilder class 
        StringBuilder sb = new StringBuilder(); 
  
        // Appends characters one by one 
        for (Character ch : str) { 
            sb.append(ch); 
        } 
  
        // convert in string 
        String string = sb.toString(); 
  
        // print string 
        System.out.println(string); 
    } 
} 
```
#### 2. 引入Google的Joiner
```java
// Java program for convert character list to string 
import com.google.common.base.Joiner; 
import java.util.Arrays; 
import java.util.List; 
  
// Convert List of Characters to String in Java 
class GFG { 
    public static void main(String[] args) 
    { 
        // create character list and initialize 
        List<Character> str = Arrays.asList('G', 'e', 'e', 'k'); 
  
        // convert in string 
        // use join() method 
        String string = Joiner.on("").join(str); 
  
        // print string 
        System.out.println(string); 
    } 
} 
```
#### 3. List.toString(), String.substring() and String.replaceAll()组合使用
```java
// Java program for convert character list to string 
import java.util.Arrays; 
import java.util.List; 
  
// Convert List of Characters to String in Java 
class GFG { 
    public static void main(String[] args) 
    { 
        // create character list and initialize 
        List<Character> str = Arrays.asList('G', 'e', 'e', 'k'); 
  
        // convert in string 
        // remove [] and spaces 
        String string = str.toString() 
                            .substring(1, 3 * str.size() - 1) 
                            .replaceAll(", ", ""); 
  
        // print string 
        System.out.println(string); 
    } 
} 
```

#### 4. collectors(Java 8 新增)
```java
// Java program for convert character list to string 
import java.util.stream.Collectors; 
import java.util.Arrays; 
import java.util.List; 
  
// Convert List of Characters to String in Java 
class GFG { 
    public static void main(String[] args) 
    { 
        // create character list and initialize 
        List<Character> str = Arrays.asList('G', 'e', 'e', 'k'); 
  
        // convert in string 
        // using collect and joining() method 
        String string =  str.stream().map(String::valueOf).collect(Collectors.joining()); 
  
        // print string 
        System.out.println(string); 
    } 
} 
```


#### 总结
方法2最类似动态语言的写法，最简单。推荐。
如果不想安装Google的Guava的话，可以选择方法4，但是必须得Java 8版本才支持。