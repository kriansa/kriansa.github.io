---
layout: page
title: Projects
description: Some of my projects
header-img: img/projects-bg.jpg
permalink: /projectss
---
{% for post in site.categories.projects %}
<div class="post-preview">
    <a href="{{ post.url | prepend: site.baseurl }}">
        <h2 class="post-title">{{ post.title }}</h2>
        {% if post.subtitle %}
        <h3 class="post-subtitle">
            {{ post.subtitle }}
        </h3>
        {% endif %}
    </a>
    <p class="post-meta">Posted on {{ post.date | date: "%B %-d, %Y" }}</p>
</div>
{% unless forloop.last %}<hr>{% endunless %}
{% endfor %}
