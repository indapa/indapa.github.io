---
layout: default
---

<h1>Posts</h1>

{% for post in site.posts %}
  <article>
    <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    <span class="date">{{ post.date | date: "%B %d, %Y" }}</span>
    {{ post.excerpt }}
  </article>
{% endfor %}