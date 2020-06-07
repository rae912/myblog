---
layout: post
title: "使用requests时候获取乱码问题的解决方法"
subtitle: "Solve the problem of messy code when use requests lib"
author: "qingshan"
header-img: "img/about-bg-walle.jpg"
header-mask: 0.4
tags:

- 工作
- Python

---

今天在使用requests提取一些网页数据的时候，发现总是乱码：
```python
import sys
print(sys.getdefaultencoding())

import chardet

response = requests.request("GET", url, headers=headers, data = payload)
print(chardet.detect(response.text.encode('utf-8')))
print(response.text)

```

结果如下：
```bash
utf-8
{'confidence': 0.99, 'encoding': 'utf-8'}
"content":"b612ććçšä¸ĺ¤´äş"
```

response.text 不管是默认，还是encode('utf-8')，使用chardet显示都是utf-8无误，而且一旦使用其他编码就会报错。所以肯定在encode这一步肯定是没有问题的。那么问题只能是在requests自己这一层了。

经过搜索，发现原来Requests会基于HTTP头部对响应的编码作出有根据的推测。当你访问 r.text 之时，Requests会使用其推测的文本编码。你可以找出 Requests 使用了什么编码，并且能够使用 r.encoding 属性来改变它。所以很有可能就是这一步推测的文本编码出错了。

那么可以可以通过r.content跳过requests自身推测编码，直接获取原始内容的bytes类型再进行自定义编码encode，这样就解决了：

```python
r = requests.get(url)
content = r.content
content_doc = str(content,'utf-8') # OR content_doc = content.decode("utf-8","ignore")
print(content_doc)
```

这样就没问题了。乱码问题解决！

