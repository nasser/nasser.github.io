---
layout: page
title: Human Readable Text
---

<table class="blog">

<tr>
  <td class="date">October 2018</td>
  <td class="title"><a href="http://ojs.decolonising.digital/index.php/decolonising_digital/article/view/PersonalComputer">A Personal Computer for Children of All Cultures, Decolonising the Digital</a></td>
</tr>
<tr>
  <td class="date">April 2018</td>
  <td class="title"><a href="https://increment.com/programming-languages/crash-course-in-compilers/">A crash course in compilers, Increment Magazine</a></td>
</tr>
<tr>
  <td class="date">April 2018</td>
  <td class="title"><a href="https://increment.com/programming-languages/unplain-text-primer-on-non-latin/">Unplain Text, Increment Magazine</a></td>
</tr>

</table>

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