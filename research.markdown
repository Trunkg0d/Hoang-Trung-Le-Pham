---
layout: academic
title: Research
permalink: /research/
---

<header class="page-intro site-container">
  <p class="eyebrow">Research</p>
  <h1>Trustworthy learning through security, privacy, and reliable forgetting.</h1>
  <p class="page-lead">My research studies what machine learning systems retain, what they can safely forget, and how those properties can be evaluated under realistic attacks.</p>
</header>

<section class="site-container research-layout">
  <div class="research-topic">
    <span class="topic-number">01</span>
    <div>
      <h2>Machine Unlearning</h2>
      <p>I work on methods that remove the influence of selected training data without requiring complete retraining. A central part of this work is evaluation: a model should not only preserve utility, but also demonstrate that forgotten information cannot be recovered.</p>
      <ul class="plain-list">
        <li>Feature-space and optimization-based unlearning</li>
        <li>Removal quality beyond predictive accuracy</li>
        <li>Auditing and reproducible evaluation protocols</li>
      </ul>
    </div>
  </div>

  <div class="research-topic">
    <span class="topic-number">02</span>
    <div>
      <h2>Model Security & Privacy</h2>
      <p>I investigate how model updates can create new attack surfaces. My current focus includes inversion attacks against unlearned models and the security implications of knowledge editing.</p>
      <ul class="plain-list">
        <li>Model inversion and information recovery</li>
        <li>Adversarial attacks on forgetting mechanisms</li>
        <li>Security analysis of knowledge editing</li>
      </ul>
    </div>
  </div>

  <div class="research-topic">
    <span class="topic-number">03</span>
    <div>
      <h2>Multimodal Learning</h2>
      <p>I also explore robust multimodal systems that combine visual, linguistic, and structured information, with an emphasis on practical applications and dependable behavior.</p>
    </div>
  </div>
</section>

<section class="site-container research-cta">
  <p>For papers, venues, and DOI links, visit the complete publication record.</p>
  <a class="button" href="{{ '/papers/' | relative_url }}">Browse publications</a>
</section>
