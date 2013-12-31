---
layout: page
title: GamingRobot's
tagline: Blog Thing
---
{% include JB/setup %}

{% for post in site.posts limit 4 %}
<h1><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h1>
<span>{{ post.date | date_to_string }}</span>
{{ post.content | strip_html | truncatewords:75}}
<br>
  <a href="{{ post.url }}">Read more...</a>
<br>
<hr>
{% endfor %}
