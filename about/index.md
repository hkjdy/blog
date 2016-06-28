---
title: 关于
layout: page
comments: no
---

{{ site.about }}

----

### 联系方式：

{% if site.qq %}
ＱＱ：[{{ site.qq }}](tencent://message/?uin={{ site.qq }})
{% endif %}

电话：

```python 

[(247*x**10-12071*x**9+251772*x**8-2926326*x**7+20757891*x**6-92496327*x**5+256503938*x**4-421551484*x**3+365509512*x**2-123134112*x+725760)/725760 for x in xrange(11)]

```

网站：[{{ site.name }}]({{ site.url }})

邮箱：[{{ site.email }}](mailto:{{ site.email }})

GitHub : [http://github.com/{{ site.github }}](http://github.com/{{ site.github }})

----

{% if site.weibo %}
[![新浪微博](http://tva3.sinaimg.cn/crop.0.0.180.180.180/6c911b05jw1e8qgp5bmzyj2050050aa8.jpg)](http://weibo.com/{{ site.weibo }})
{% endif %}
