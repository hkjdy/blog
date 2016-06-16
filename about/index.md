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
网站：[{{ site.name }}]({{ site.url }})

邮箱：[{{ site.email }}](mailto:{{ site.email }})

GitHub : [http://github.com/{{ site.github }}](http://github.com/{{ site.github }})

----

{% if site.weibo %}
[![新浪微博](http://tva3.sinaimg.cn/crop.0.0.180.180.180/6c911b05jw1e8qgp5bmzyj2050050aa8.jpg)](http://weibo.com/{{ site.weibo }})
{% endif %}
