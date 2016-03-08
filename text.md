---
layout: page
title: Human Readable Text
---

<table class="blog">
{% for post in site.posts %}
{% unless post.private %}
  <tr>
    <td class="date">{{ post.date | date_to_string}}</td>
<td class="title"><a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></td>
  </tr>
{% endunless %}
{% endfor %}
</table>