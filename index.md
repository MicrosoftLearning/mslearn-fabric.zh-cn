---
title: 联机托管说明
permalink: index.html
layout: home
---

# Microsoft Fabric 练习

以下练习旨在支持 [Microsoft Learn](https://aka.ms/learn-fabric) 上的模块。

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| 模块 | 实验室 |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

