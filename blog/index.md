---
layout: default
title: Blog
---

<!-- Add favicon for blog page -->
<link rel="icon" type="image/x-icon" href="/images/Bonny.png" />

<!-- Blog List View -->
<div id="blog-list-view" class="blog-list" style="max-width:800px;margin:2em auto 3em auto;">
  {% for post in site.posts %}
    <div class="blog-post-item card mb-4" style="padding:1.5em;display:flex;align-items:center;gap:1.5em;">
      {% if post.image %}
        <img src="{{ post.image }}" alt="{{ post.title }}" class="img-fluid blog-thumb" style="max-width:100px;border-radius:8px;" />
      {% endif %}
      <div style="flex:1;">
        <h2 style="font-size:1.3rem;margin-bottom:0.5rem;"><a href="{{ post.url }}">{{ post.title }}</a></h2>
        <div class="blog-date mb-2" style="color:#888;">{{ post.date | date: "%B %d, %Y" }}</div>
        <div class="blog-desc">{{ post.excerpt | strip_html | truncatewords: 30 }}</div>
        <a href="{{ post.url }}" class="btn btn-sm btn-primary mt-2">Read more</a>
      </div>
    </div>
  {% endfor %}
</div>
