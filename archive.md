---
layout: page
title: 文章归档
permalink: /archive/
---

# 所有文章

{% for post in site.posts %}
## [{{ post.title }}]({{ post.url }})
_{{ post.date | date: "%Y年%m月%d日" }}_
{% for category in post.categories %}
  <span class="category">{{ category }}</span>
{% endfor %}
{% endfor %}