---
layout: default
title: Blog
---

<!-- Add favicon for blog page -->
<link rel="icon" type="image/x-icon" href="../images/Bonny.png" />

<!-- Mobile-optimized blog list CSS -->
<style>
@media (max-width: 600px) {
  .blog-list {
    max-width: 100% !important;
    padding: 0 !important;
  }
  .blog-post-item {
    display: flex !important;
    flex-direction: column !important;
    align-items: stretch !important;
    justify-content: flex-start !important;
    gap: 0 !important;
    padding: 0 !important;
    margin: 0 0 1.5em 0 !important;
    border-radius: 10px !important;
    box-shadow: 0 2px 8px rgba(0,0,0,0.05) !important;
    background: #fff !important;
  }
  .blog-post-item img.blog-thumb {
    display: block !important;
    width: 100% !important;
    max-width: 100% !important;
    height: auto !important;
    margin: 0 0 0.7em 0 !important;
    border-radius: 8px 8px 0 0 !important;
    object-fit: cover !important;
    box-shadow: 0 1px 4px rgba(0,0,0,0.08) !important;
  }
  .blog-post-item > div {
    width: 100% !important;
    text-align: left !important;
    padding: 0 1em 1em 1em !important;
  }
  .blog-post-item h2 {
    font-size: 1.1rem !important;
    margin: 0.5em 0 0.3em 0 !important;
  }
  .blog-desc {
    font-size: 0.98rem !important;
    margin-bottom: 0.5em !important;
  }
  .blog-date {
    font-size: 0.9rem !important;
    margin-bottom: 0.3em !important;
  }
}
</style>


<!-- Blog List View -->
<div id="blog-list-view" class="blog-list" style="max-width:800px;margin:2em auto 3em auto;">
  {% for post in site.posts %}
    <div class="blog-post-item card mb-4" style="padding:1.5em;display:flex;align-items:center;gap:1.5em;">
      {% if post.image %}
        <img src="{{ post.image }}" alt="{{ post.title }}" class="img-fluid blog-thumb" style="max-width:200px;width:100%;height:auto;border-radius:8px;" />
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
