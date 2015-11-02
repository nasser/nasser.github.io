---
layout: page
title: Blog
---

<table class="blog">
{% for post in site.posts %}
  <tr>
    <td class="date">{{ post.date | date_to_string}}</td>
<td class="title"><a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></td>
  </tr>
{% endfor %}
</table>