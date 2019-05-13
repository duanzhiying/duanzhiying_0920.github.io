---
layout: page
title: About
description: 瑞雪兆丰年
keywords: kidzy,duanzhiying
comments: true
menu: 关于
permalink: /about/
---

## 伴随我名字的我的名字：
`瑞雪兆丰年`


## 伴随我信念的我的信念：
`全世界都需要你`


## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}

***
未完待续……
