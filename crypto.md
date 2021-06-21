---
layout: default
---
<ul>
  {% for post in site.posts reversed %}
    {% if post.categories contains 'cryptography' %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
    {% endif %}
  {% endfor %}
</ul>
