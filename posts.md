---
layout: page
title: Posts
---

{% for post in site.posts %}
    {% if post.project == null %}
<ul>
    <li>
        <a class="zoombtn" href="{{ site.url }}{{ post.url }}">{{ post.title }}</a>
        <p>{{ post.excerpt }}</p>
    </li>
</ul>
    {% endif %}
{% endfor %}
