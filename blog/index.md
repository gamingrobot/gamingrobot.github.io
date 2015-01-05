---
layout: page
title: Morgan Creekmore's
tagline:
---
{% include JB/setup %}

{% for post in site.posts limit 4 %}
<h1><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h1>
<span>{{ post.date | date_to_string }}</span>
{{ post.excerpt }}
  <a href="{{ post.url }}">Read more...</a>
<br>
<hr>
{% endfor %}
