---
layout: default
title: Writing
description: Research notes by Hoang-Trung Le-Pham.
permalink: /writing/
---

<header class="page-hero">
  <div class="shell page-hero-grid">
    <div>
      <p class="eyebrow">Research notes</p>
      <h1>Writing</h1>
    </div>
    <p class="page-summary">Long-form technical writing that connects research concepts, threat models, and practical implications.</p>
  </div>
</header>

<section class="shell writing-list">
  {%- assign post = site.posts | first -%}
  <article class="writing-card">
    <div class="writing-cover">
      <img src="{{ '/images/machine-unlearning/mu-vs-inversion-attacks.png' | relative_url }}" alt="Machine unlearning and inversion attacks diagram">
    </div>
    <div class="writing-copy">
      <span class="article-date">{{ post.date | date: "%B %-d, %Y" }} · 18 min read</span>
      <h2>{{ post.title }}</h2>
      <p>{{ post.description }}</p>
      <a class="button button-primary" href="{{ post.url | relative_url }}">Read the article <span aria-hidden="true">→</span></a>
    </div>
  </article>
</section>
