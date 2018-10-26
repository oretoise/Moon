---
layout: page
title: Posts
---

<div class="post-list">
    {% for post in site.posts %}
        <div class="post-item">
            <a class="zoombtn" href="{{ site.url }}{{ post.url }}"><h2>{{ post.title }}</h2></a>
            <span class="post-date">{{ post.date | date_to_string }}</span>
            <p>{{ post.excerpt }}</p>
        </div>
    {% endfor %}
</div>
