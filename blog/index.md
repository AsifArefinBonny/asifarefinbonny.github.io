---
layout: default
title: Blog
---

<!-- Add favicon for blog page -->
<link rel="icon" type="image/x-icon" href="/images/Bonny.png" />

<!-- Blog View Toggle and Home Button (styled to match main navbar, placed at top) -->
<div id="blog-navbar-controls" style="display:flex;align-items:center;justify-content:flex-end;padding:1em 0 0.5em 0;gap:1em;max-width:900px;margin:0 auto;">
  <button id="blog-list-toggle" class="btn btn-primary" style="min-width:120px;">List View</button>
  <button id="blog-detail-toggle" class="btn btn-secondary" style="min-width:120px;">Detail View</button>
  <a href="/index.html" class="btn btn-outline-primary">&larr; Home</a>
</div>

<h1 style="text-align:center;">Blog</h1>

<!-- Blog List View -->
<div id="blog-list-view" class="blog-list" style="max-width:800px;margin:2em auto 3em auto;">
  {% for post in site.posts %}
    <div class="blog-post-item card mb-4" style="padding:1.5em;display:flex;align-items:center;gap:1.5em;">
      {% if post.image %}
        <img src="{{ post.image }}" alt="{{ post.title }}" class="img-fluid blog-thumb" style="max-width:100px;border-radius:8px;" />
      {% endif %}
      <div style="flex:1;">
        <h2 style="font-size:1.3rem;margin-bottom:0.5rem;"><a href="#" class="blog-detail-link" data-url="{{ post.url }}">{{ post.title }}</a></h2>
        <div class="blog-date mb-2" style="color:#888;">{{ post.date | date: "%B %d, %Y" }}</div>
        <div class="blog-desc">{{ post.excerpt | strip_html | truncatewords: 30 }}</div>
        <a href="#" class="btn btn-sm btn-primary mt-2 blog-detail-link" data-url="{{ post.url }}">Read more</a>
      </div>
    </div>
  {% endfor %}
</div>

<!-- Blog Detail View (hidden by default) -->
<div id="blog-detail-view" style="display:none;max-width:800px;margin:2em auto 3em auto;"></div>

<!-- Blog Footer -->
<footer class="blog-footer" style="text-align:center;padding:2em 0 1em 0;border-top:1px solid #eee;margin-top:3em;">
  <div style="font-weight:600;font-size:1.1em;">Asif Arefin Bonny</div>
  <div style="margin:0.5em 0;">
    <a href="mailto:asifarefinbonny@gmail.com" style="margin:0 0.5em;">asifarefinbonny@gmail.com</a>
    <a href="https://github.com/AsifArefinBonny" target="_blank" style="margin:0 0.5em;">GitHub</a>
  </div>
</footer>

<script>
// Blog view toggler
const listView = document.getElementById('blog-list-view');
const detailView = document.getElementById('blog-detail-view');
const listBtn = document.getElementById('blog-list-toggle');
const detailBtn = document.getElementById('blog-detail-toggle');

function showListView() {
  listView.style.display = '';
  detailView.style.display = 'none';
  listBtn.classList.add('btn-primary');
  listBtn.classList.remove('btn-secondary');
  detailBtn.classList.remove('btn-primary');
  detailBtn.classList.add('btn-secondary');
}
function showDetailView(html) {
  listView.style.display = 'none';
  detailView.style.display = '';
  detailView.innerHTML = html;
  listBtn.classList.remove('btn-primary');
  listBtn.classList.add('btn-secondary');
  detailBtn.classList.add('btn-primary');
  detailBtn.classList.remove('btn-secondary');
}
listBtn.onclick = showListView;
detailBtn.onclick = function() {
  // If no post loaded, just show list
  if (!detailView.innerHTML) showListView();
  else showDetailView(detailView.innerHTML);
};
// Use event delegation for blog post links
listView.addEventListener('click', function(e) {
  const link = e.target.closest('.blog-detail-link');
  if (link) {
    e.preventDefault();
    const url = link.getAttribute('data-url');
    fetch(url)
      .then(r => r.text())
      .then(html => {
        // Extract the main article from the post page
        const temp = document.createElement('div');
        temp.innerHTML = html;
        const article = temp.querySelector('.blog-post');
        if (article) {
          // Add a back button
          article.insertAdjacentHTML('afterbegin', '<a href="#" class="btn btn-outline-primary mb-3" id="back-to-list">&larr; Back to List</a>');
          detailView.innerHTML = '';
          detailView.appendChild(article);
          showDetailView(detailView.innerHTML);
          document.getElementById('back-to-list').onclick = function(ev) {
            ev.preventDefault();
            showListView();
          };
        }
      });
  }
});
// Default to list view
showListView();
</script>
