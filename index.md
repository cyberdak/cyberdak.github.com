---
layout: page
title: 首页
tagline: My Epic Life
---
{% include JB/setup %}

<img src="https://i.loli.net/2017/09/10/59b4da5daf0ab.jpg" alt="OVER 3500 INVESTOR (1).jpg" title="OVER 3500 INVESTOR.jpg" />
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
