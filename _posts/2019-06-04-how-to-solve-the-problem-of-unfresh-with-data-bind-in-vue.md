---
layout: post
title: "解决Vue中数据绑定无法刷新视图的问题"
subtitle: "Solve the problem of unfresh view in Vue"
author: "qingshan"
header-img: "img/post-bg-js-version.jpg"
header-mask: 0.4
tags:
  - 工作
  - 前端
  - Vue
---

Vue.js等现代前端框架的`双向数据绑定`一直是开发者津津乐道的重要功能。依靠动态的数据绑定，很多控件可以轻松的实现功能。

今天在使用iView的`Input`组件时，发现有个控件的值改变了，但是并不能立刻在试图中刷新，而是要在其他的逻辑中触发刷新或者值重载才可以生效。原来以为是iView组件或者Vue.js的Bug，在github上的issue界面检索也发现有人遇到了同样的问题。但是却没有人提出解决方案。经过大量的搜索和查阅相关资料，终于知道了root cause所在。

### Root Cause
Vue中的数据绑定基于变量的更新检测，如果变量的值发生变化而Vue无法检测，自然就无法更新试图。

### 详细解释
在Vue的开发者文档中，有如下的介绍：

> Vue包装了数个数组操作函数，使用这些方法操作的数组去，其数据变动时会被vue监测： 
>* push()
>* pop()
>* shift()
>* unshift()
>* splice()
>* sort()
>* reverse()
>* filter(), concat(), slice() 。这些不会改变原始数组，但总是返回一个新数组。当使用非变异方法时，可以用新数组替换旧数组


> Vue 不能检测以下变动的数组： 
>* ① 当你利用索引直接设置一个项时，vm.items[indexOfItem] = newValue
>* ② 当你修改数组的长度时，例如： vm.items.length = newLength

### 解决方法
vue中有个方法可以观测Vue.set(items, indexOfItem, newValue)
![](https://ae01.alicdn.com/kf/HTB1dwBUbHus3KVjSZKb760qkFXap.png)


### 拓展 -- JavaScript中的深拷贝（deepcopy）
在开发前端过程当中，遇到一些需要复制控件的场景，就需要用到深拷贝。否则就是浅拷贝（引用），会出现一个值变化，所有拷贝的地方的值都会随着变化。
深拷贝的方法，就是另起一个变量，将值复制到中间变量，再复制到新的变量。例如：
```javascript
function copy (obj) {
   let newObj = {};
     for (let item in obj ){
       newObj[item] = obj[item]
     }
     return newObj;
}
 
var copyObj = copy(obj);
```

