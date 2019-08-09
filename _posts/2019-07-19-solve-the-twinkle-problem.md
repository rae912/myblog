---
layout: post
title: "解决Vue页面闪烁问题"
subtitle: "Solve the twinkle problem in Vue.js"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - 工作
  - Vue
---

使用Vue编写单页面应用时，有这么一个div:
```html
  <div class="ui segment" v-if="showEdit">
  ...
  </div>
```
 showEdit初始值是false。我希望加载时不显示这个div。但是实际情况是，页面刷新时，这个页面会先出现，再很快的又消失了。给用户的体验是“页面闪了”一下。

 搜索了一大圈，大部分的解决方案是使用`v-cloak`配合CSS方案，但是在我这里没有生效。原因未知。

 最后在前端妹子启发下，采用`v-show`解决了这个问题，具体如下：
```html
  <div class="ui segment" v-show="showEdit" style="display:none">
  ...
  </div>
```
原理是：v-show与v-if不同，v-if是默认不渲染该标签，直到遇到表达式为true时才渲染。所以理论上v-if不应该出现闪烁的情况，但是实际上无法解释为何会出现。v-show是默认就渲染该标签的，遇到后面CSS配置“不显示”时候，就可以直接隐藏该标签。

最后，通过v-show配合CSS解决了这个问题。