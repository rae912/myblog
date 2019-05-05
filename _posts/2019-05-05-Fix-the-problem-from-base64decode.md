---
layout: post
title: "Python base64解码时报错的解决方法"
subtitle: "Fix the problem from python base64 decoding"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - 工作
  - Python
---

最近在使用Python写一个小工具，需要用到base64解码。示例代码如下：
```python
data = base64.b64decode(base64text)
```
收到报错提示：
```python
  File "/xxx/2.7/lib/python2.7/base64.py", line 76, in b64decode
    raise TypeError(msg)
TypeError: Incorrect padding
```

将base64内容放在在线解码尝试解码，又没有任何问题。所以问题还是出现在python这边。根据错误提示**TypeError: Incorrect padding**进行搜索，得到这样的提示：
```shell
输入的base64编码字符串必须符合base64的padding规则。“当原数据长度不是3的整数倍时, 如果最后剩下两个输入数据，在编码结果后加1个“=”；
如果最后剩下一个输入数据，编码结果后加2个“=”；如果没有剩下任何数据，就什么都不要加，这样才可以保证资料还原的正确性。
```

因此，这说明原始的base64没有满足这个padding规范，需要使用规则判断一下，不满足padding规范的就修正直到复合。

hotfix代码
```python
try:
    if divmod(len(field),4)[1] != 0:
        field += "="*(4-divmod(len(field),4)[1])
    #decode field here
except Exception,e:
    print(e)
```

或者直接用优化后的base64decode方法
```python
def decode_base64(data):
    """Decode base64, padding being optional.

    :param data: Base64 data as an ASCII byte string
    :returns: The decoded byte string.

    """
    missing_padding = 4 - len(data) % 4
    if missing_padding:
        data += b'='* missing_padding
    return base64.decodestring(data)
```

#### 其他相关问题
另需要注意的是，除了padding问题需要注意以外，如果被编码的字符串是URL格式，需要使用专用的**base64.urlsafe_b64decode**方法来进行解码。它可以避免URL中的参数被错误的也按照base64格式解码，否则可能也会有相同的padding错误提示。
进一步优化后的方法如下：
```python
def decode_base64(data):
    """Decode base64, padding being optional.

    :param data: Base64 data as an ASCII byte string
    :returns: The decoded byte string.

    """
    data = data.replace("\r", "").replace("\n", "").replace("\t", "")
    missing_padding = 4 - len(data) % 4
    if missing_padding:
        data += b'='* missing_padding

    try:
    	result = base64.urlsafe_b64decode(data)
    except Exception, e:
    	result = base64.b64decode(data)

    return result
```




