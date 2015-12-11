---
title: 关于
layout: page
comments: no
---

---

###自我介绍

{{ site.about }}

专业Android程序员，写过Java/Python后台，做过H5前端...

> 1. 掌握技能：Android SDK、Android NDK（JNI）；Spring MVC、Mysql；HTML、CSS、JS。 
2. 其他使用过的技能：Linux 简单运维、C++、 Python。 
3. 注重编程规范，崇尚简单美，一直致力于写简单高效的代码。 
4. 喜欢逛开源社区，喜欢研究开源项目，喜欢写技术文档和写博客分享。 
5. 一年的 Android 团队组长、一年的技术团队负责人的经验。 
6. 热爱运动，热爱学习，善于团队合作。

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
