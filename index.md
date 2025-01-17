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

# Recent GitHub Activity

<div id="github-activity"></div>

<script>
  fetch('https://github.com/indapa.atom')
    .then(response => response.text())
    .then(str => new window.DOMParser().parseFromString(str, "text/xml"))
    .then(data => {
      const items = data.querySelectorAll('entry');
      const activityHtml = Array.from(items).slice(0, 10).map(item => {
        const title = item.querySelector('title').textContent;
        const link = item.querySelector('link').getAttribute('href');
        const date = new Date(item.querySelector('published').textContent);
        return `<p><a href="${link}">${title}</a><br>
                <small>${date.toLocaleDateString()}</small></p>`;
      }).join('');
      document.getElementById('github-activity').innerHTML = activityHtml;
    });
</script>
