---
layout: page
title: Tutorials
permalink: /tutorials
---
<!-- Display ChE posts  -->
<div class="tag-section">
  <h2>Reaction Engineering</h2>
  <div class="blog-posts">
    {% assign tag_posts = site.posts | where: "tags", "rxn" %}
    {% for post in tag_posts %}
      <div class="blog-post">
        <a href="{{ post.url }}">
          <div class="blog-post-image">
            {% if post.image %}
              <img src="{{ post.image }}" alt="{{ post.title }}">
            {% endif %}
          </div>
          <div class="blog-post-info">
            <h2>{{ post.title }}</h2>
            <small>{{ post.date | date: "%B %d, %Y" }}</small>
          </div>
        </a>
      </div>
    {% endfor %}
  </div>
</div>

<!-- Display FT posts  -->
<div class="tag-section">
  <h2>Fourier Transform</h2>
  <div class="blog-posts">
    {% assign tag_posts = site.posts | where: "tags", "fourier" %}
    {% for post in tag_posts %}
      <div class="blog-post">
        <a href="{{ post.url }}">
          <div class="blog-post-image">
            {% if post.image %}
              <img src="{{ post.image }}" alt="{{ post.title }}">
            {% endif %}
          </div>
          <div class="blog-post-info">
            <h2>{{ post.title }}</h2>
            <small>{{ post.date | date: "%B %d, %Y" }}</small>
          </div>
        </a>
      </div>
    {% endfor %}
  </div>
</div>