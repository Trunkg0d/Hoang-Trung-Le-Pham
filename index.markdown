---
layout: default
title: Hoang-Trung Le-Pham — AI Researcher
description: AI researcher working on trustworthy AI, machine unlearning, model security, and multimodal learning.
---

<section class="hero">
  <div class="shell hero-grid">
    <div>
      <p class="eyebrow">AI Researcher · Ho Chi Minh City</p>
      <h1 class="display-title">Building AI that<br><span class="accent">earns trust.</span></h1>
      <p class="hero-lead">
        I'm <strong>Hoang-Trung Le-Pham</strong>, a teaching assistant at the University of
        Information Technology, VNU-HCM, and an AI researcher with the IEC Research Group (led by
        Associate Professor Kim-Hung Le). My work explores model security, machine unlearning, and multimodal learning.
      </p>
      <div class="hero-actions">
        <a class="button button-primary" href="{{ '/papers/' | relative_url }}">Explore my research <span aria-hidden="true">→</span></a>
        <a class="button" href="mailto:trunglph@uit.edu.vn">Get in touch</a>
      </div>
      <div class="hero-meta">
        <span>MSc in Artificial Intelligence · expected 2026</span>
        <span>IEC Research Group</span>
      </div>
    </div>

    <div class="portrait-card">
      <div class="portrait-frame">
        <img src="{{ '/images/profile.jpg' | relative_url }}" alt="Portrait of Hoang-Trung Le-Pham" width="852" height="852">
      </div>
      <div class="availability">Open to PhD opportunities in trustworthy AI</div>
    </div>
  </div>
</section>

<section class="stats" aria-label="Research highlights">
  <div class="shell stats-grid">
    <div class="stat"><strong>06</strong><span>Published & accepted papers</span></div>
    <div class="stat"><strong>02</strong><span>Q1 journal articles</span></div>
    <div class="stat"><strong>03</strong><span>First-author papers</span></div>
  </div>
</section>

<section class="section section-soft">
  <div class="shell">
    <div class="section-head">
      <div>
        <p class="eyebrow">Research focus</p>
        <h2 class="section-title">Trustworthy intelligence, from models to systems.</h2>
      </div>
      <p class="section-intro">I study how AI systems can remain useful while becoming safer, more private, and more robust in the real world.</p>
    </div>

    <div class="focus-grid">
      <article class="focus-card">
        <!-- <span class="focus-index">01</span> -->
        <h3>Machine Unlearning</h3>
        <p>Removing the influence of selected training data while preserving model utility and measuring what the model still reveals.</p>
      </article>
      <article class="focus-card">
        <!-- <span class="focus-index">02</span> -->
        <h3>AI Model Security</h3>
        <p>Investigating inversion, dropout, and bit-flip attacks—and designing stronger evaluation and defense mechanisms.</p>
      </article>
      <article class="focus-card">
        <!-- <span class="focus-index">03</span> -->
        <h3>Multimodal Learning</h3>
        <p>Combining semantic, graph, interaction, and temporal signals for robust early rumor detection on social media.</p>
      </article>
    </div>
  </div>
</section>

<section class="section">
  <div class="shell">
    <div class="section-head">
      <div>
        <p class="eyebrow">Selected research</p>
        <h2 class="section-title">Recent work and ideas.</h2>
      </div>
      <a class="text-link" href="{{ '/papers/' | relative_url }}">All publications <span aria-hidden="true">→</span></a>
    </div>

    <div class="featured-work">
      <article class="feature-panel">
        <p class="eyebrow">Information Fusion · 2026 · First author</p>
        <h3>MTAS: A Temporal-Aware Multiview Graph Attention Framework for Early Rumor Detection</h3>
        <p>A multiview framework that brings semantic, interaction, graph, and temporal information together for earlier and more accurate rumor detection.</p>
        <a class="text-link" href="https://doi.org/10.1016/j.inffus.2025.103598" target="_blank" rel="noreferrer">Read the paper <span aria-hidden="true">↗</span></a>
      </article>

      <div class="feature-side">
        <article class="mini-panel">
          <span class="label">Array · Q1</span>
          <h3>ODFU: Federated unlearning for dropout attack defense</h3>
          <a class="text-link" href="https://doi.org/10.1016/j.array.2026.101065" target="_blank" rel="noreferrer">View publication <span aria-hidden="true">↗</span></a>
        </article>
        <article class="mini-panel">
          <span class="label">ATC 2026 · Accepted</span>
          <h3>FLIA: Forgotten label inference attacks against machine unlearning</h3>
          <a class="text-link" href="{{ '/papers/' | relative_url }}#flia">View details <span aria-hidden="true">→</span></a>
        </article>
      </div>
    </div>
  </div>
</section>

<section class="section section-soft">
  <div class="shell home-writing">
    <div>
      <p class="eyebrow">Featured writing</p>
      <h2 class="section-title">Research, unpacked.</h2>
      <p class="section-intro">A single, focused guide to the privacy gap between machine unlearning and inversion attacks.</p>
    </div>

    {%- assign writing_posts = site.posts | sort: "writing_order" -%}
    {%- assign featured_post = writing_posts | first -%}
    {%- assign featured_origin = featured_post.content_origin | default: "human" -%}
    {%- assign featured_origin_label = featured_post.content_origin_label | default: "Human" -%}
    <article class="article-preview">
      <div class="article-preview-content">
        <div class="article-meta-row">
          <span class="article-date">{{ featured_post.date | date: "%B %Y" }} · {{ featured_post.reading_time }}</span>
          <span class="content-label content-label--{{ featured_origin }}" title="{{ featured_post.content_origin_note | default: 'Written without generative AI' }}">{{ featured_origin_label }}</span>
        </div>
        <h3>{{ featured_post.title }}</h3>
        <p>{{ featured_post.description }}</p>
        <a class="text-link" href="{{ featured_post.url | relative_url }}">Read article <span aria-hidden="true">→</span></a>
      </div>
    </article>
  </div>
</section>
