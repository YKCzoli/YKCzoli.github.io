---
layout: page
title: Previously
permalink: /posts/
---


  <ul class="post-list">
    {% for post in site.posts %}
      <li>

        <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>

        <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>

      </li>
    {% endfor %}
  </ul>
