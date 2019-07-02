---
layout: post
title: "iView组件额外传参的方法"
subtitle: "The way for iview pass external parameter"
author: "qingshan"
header-img: "img/post-bg-js-version.jpg"
header-mask: 0.4
tags:
  - 工作
  - iview
  - 前端
---

今天在一个前端需求中遇到一个问题：需要在iview控件的事件中传递一个自定义参数。但是默认情况下，iview的控件是已经定义好的，如果不填写参数，则传递默认参数；如果填上参数，传递的还是默认参数。例如：
```javascript
<Select v-model="item.type" @on-change="set_parse_type">
  <Option value="0">内网</Option>
  <Option value="1">办公网</Option>
  <Option value="2">公网</Option>
</Select>


methods:{
	set_parse_type(value){
		console.log(value);
	}

```
查看文档，描述如下：

|事件名	|说明|	返回值|
|----|----|----|
|on-change|	选中的Option变化时触发，默认返回 value，如需返回 label，详见 label-in-value 属性|	当前选中项|

但是需求是除了value以外，还需要传递一个自定义参数item.index。最终找到解决办法：
*on-change事件是有默认返回值event的，$event即表示on-change时间默认的返回的参数*，所以可以用$event代表内置的参数，其他参数代表自定义参数：
```javascript
<Select v-model="item.type" @on-change="set_parse_type(item.index, $event)">
  <Option value="0">内网</Option>
  <Option value="1">办公网</Option>
  <Option value="2">公网</Option>
</Select>

methods:{
	set_parse_type(index, event){
		console.log(index, event);
	}
```

