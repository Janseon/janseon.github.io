---
title: 关于
layout: page
comments: no
---

---

###自我介绍

{{ site.about }}

专业Android程序员，写过Java/Python后台，做过H5前端...

> 崇尚简单美

----

###联系方式：

{% if site.qq %}
ＱＱ：[{{ site.qq }}](tencent://message/?uin={{ site.qq }})
{% endif %}
网站：[{{ site.name }}]({{ site.url }})

邮箱：[{{ site.email }}](mailto:{{ site.email }})

GitHub：[http://github.com/{{ site.github }}](http://github.com/{{ site.github }})

{% if site.cnblogs %}
@cnblogs：{{ site.cnblogs }}
{% endif %}