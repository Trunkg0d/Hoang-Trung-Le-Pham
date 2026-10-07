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
    <div>
      <p class="page-summary">Long-form technical writing that connects research concepts, threat models, and practical implications.</p>
      <p class="content-label-guide"><strong>Label key:</strong> Slop = AI-written · Near Slop = AI-assisted and human-edited · Human = written without generative AI</p>
    </div>
  </div>
</header>

<section class="shell writing-list">
  {%- assign post = site.posts | first -%}
  {%- assign post_origin = post.content_origin | default: "human" -%}
  {%- assign post_origin_label = post.content_origin_label | default: "Human" -%}
  <article class="writing-card">
    <div class="writing-copy">
      <div class="article-meta-row">
        <span class="article-date">{{ post.date | date: "%B %-d, %Y" }} · {{ post.reading_time }}</span>
        <span class="content-label content-label--{{ post_origin }}" title="{{ post.content_origin_note | default: 'Written without generative AI' }}">{{ post_origin_label }}</span>
      </div>
      <h2>{{ post.title }}</h2>
      <p>{{ post.description }}</p>
      <a class="button button-primary" href="{{ post.url | relative_url }}">Read the article <span aria-hidden="true">→</span></a>
    </div>
  </article>
</section>
