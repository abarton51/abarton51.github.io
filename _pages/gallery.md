---
layout: archive
title: "Photo Gallery"
permalink: /gallery/
author_profile: true
---

{% include base_path %}

<div class="gallery-index">
  {% for post in site.gallery %}
    <div class="gallery-item">
      <a href="{{ base_path }}{{ post.url }}" class="gallery-link">
        <div class="gallery-image-container">
          <img src="{{ base_path }}/images/gallery/{{ post.header.teaser }}" alt="{{ post.title }}" class="gallery-image">
        </div>
        <h2 class="gallery-title">{{ post.title }}</h2>
      </a>
    </div>
  {% endfor %}
</div>

<style>
  .gallery-index {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    grid-gap: 20px;
    margin-top: 2em;
  }
  
  .gallery-item {
    position: relative;
    overflow: hidden;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    transition: transform 0.3s ease, box-shadow 0.3s ease;
  }
  
  .gallery-item:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 16px rgba(0,0,0,0.2);
  }
  
  .gallery-image-container {
    width: 100%;
    height: 200px;
    overflow: hidden;
  }
  
  .gallery-image {
    width: 100%;
    height: 100%;
    object-fit: cover;
    transition: transform 0.5s ease;
  }
  
  .gallery-item:hover .gallery-image {
    transform: scale(1.05);
  }
  
  .gallery-title {
    padding: 10px;
    margin: 0;
    font-size: 1.2em;
    text-align: center;
    background: rgba(0,0,0,0.7);
    color: white;
    position: absolute;
    bottom: 0;
    width: 100%;
  }
  
  .gallery-link {
    text-decoration: none;
    color: inherit;
    display: block;
    height: 100%;
  }
</style>

