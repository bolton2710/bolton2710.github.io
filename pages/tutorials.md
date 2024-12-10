---
layout: page
title: Tutorials
permalink: /tutorials
---
<!-- Display ChE posts  -->
<div class="tag-section">
  <h2>Chemical Engineering Interactive Simulations</h2>
  <div class="blog-posts">
    {% assign tag_posts = site.posts | where: "tags", "che" %}
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

<!-- Display Other posts  -->
<!-- <div class="tag-section">
  <h2>Random tutorials</h2>
  <div class="blog-posts">
    {% assign tag_posts = site.posts | where: "tags", "other" %}
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
</div> -->