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
        <a href="{{ site.url }}{{ post.url }}" class="btn zoombtn">Read More</a>
    </li>
</ul>
    {% endif %}
{% endfor %}
