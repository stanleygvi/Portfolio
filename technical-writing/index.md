---
layout: technical-page
title: Technical Writing
subtitle: Projects, security findings, CTF walkthroughs, and technical experiences.
description: Technical writing index with category navigation and recent posts.
---

<section class="section">
  <h2>Sections</h2>
  <div class="grid">
    <div class="card">
      <h3><a href="{{ '/technical-writing/project-writeups/' | relative_url }}">Project Write-Ups</a></h3>
      <p>Implementation notes, architecture decisions, and postmortems.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/technical-writing/security-findings/' | relative_url }}">Security Findings</a></h3>
      <p>Issue analysis, impact, and remediation documentation.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/technical-writing/ctf-walkthroughs/' | relative_url }}">CTF Walkthroughs</a></h3>
      <p>Challenge prompts, solve paths, and key lessons.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/technical-writing/technical-experiences/' | relative_url }}">Technical Experiences</a></h3>
      <p>Reflections on engineering execution and outcomes.</p>
    </div>
  </div>
</section>

<section class="section">
  <h2>Recent Posts</h2>
  <div class="card">
    <ul class="list">
      {% for post in site.posts limit: 8 %}
        <li>
          <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
          <span class="item-meta">({{ post.date | date: "%Y-%m-%d" }})</span>
        </li>
      {% endfor %}
    </ul>
  </div>
</section>
