---
layout: default
---
<ul>
  {% for post in site.posts reversed %}
    {% if post.categories contains 'books' %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
    {% endif %}
  {% endfor %}
</ul>
