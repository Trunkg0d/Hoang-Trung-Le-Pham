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
  {%- assign writing_posts = site.posts | sort: "writing_order" -%}
  <div class="writing-grid">
    {%- for post in writing_posts -%}
    {%- assign post_origin = post.content_origin | default: "human" -%}
    {%- assign post_origin_label = post.content_origin_label | default: "Human" -%}
    {%- assign post_status = post.status | default: "published" -%}
    <article class="writing-card writing-card--{{ post_status }}">
      <div class="writing-card-head">
        <span class="writing-index" aria-hidden="true">0{{ forloop.index }}</span>
        <div class="writing-card-badges">
          {%- if post.status_label -%}<span class="post-status">{{ post.status_label }}</span>{%- endif -%}
          <span class="content-label content-label--{{ post_origin }}" title="{{ post.content_origin_note | default: 'Written without generative AI' }}">{{ post_origin_label }}</span>
        </div>
      </div>
      <div class="writing-copy">
        <span class="article-date">{{ post.date | date: "%B %-d, %Y" }} · {{ post.reading_time }}</span>
        <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
        <p>{{ post.description }}</p>
      </div>
      <a class="writing-card-link" href="{{ post.url | relative_url }}">
        {%- if post_status == "in-progress" -%}View progress{%- else -%}Read the article{%- endif -%}
        <span aria-hidden="true">→</span>
      </a>
    </article>
    {%- endfor -%}
  </div>
</section>
