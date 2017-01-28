---
layout: default
title: Home
---

<div class="posts">
  {% for post in site.posts %}
  <div class="post">
    <h1 class="post-title">
      <a href="{{ post.url }}">
        {{ post.title }}
      </a>
    </h1>

    <span class="post-date">{{ post.date | date: "%B %-d, %Y" }}</span>

    {{ post.excerpt }}

    <p><a href="{{ post.url }}">Continue Reading &rarr;</a></p>
  </div>
  {% endfor %}
</div>
