---
# layout: default
---

<div class="blog-list">
  {% for post in site.posts %}
    <div class="blog-post-item">
      {% if post.image %}
        <img src="{{ post.image }}" alt="Blog image">
      {% endif %}
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      <p class="blog-desc">{{ post.excerpt | strip_html | truncatewords: 30 }}</p>
      <p class="blog-date">{{ post.date | date: "%B %d, %Y" }}</p>
    </div>
  {% endfor %}
</div>
