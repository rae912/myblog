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

在Vue中使用iview组件时候，一些按钮默认会引入快捷键操作。例如在确认按钮上就绑定了Enter键触发点击确认。但是实践中，发现只有第一次点击Enter键会正确触发确认，第二次及后续的急键操作都会刷新当前页面。引起不必要的麻烦。
下面就是解决这个问题的方法：

给Form表单组件的根元素绑定一个enter事件，keydown.native.enter.prevent表示prenvent 不会去 出处理DOM事件的细节，keyDownEvent写一个空方法即可

```html
<Form ref="searchFilterDiv" :model="searchFilter" :rules="ruleValidate" inline @keydown.native.enter.prevent="keyDownEvent">
</Form>
```

```javascript
keyDownEvent() {
	// 此处留空
}
```