---
layout: technical-page
title: Project Write-Ups
subtitle: Project implementation details, architecture decisions, and retrospectives.
description: Project write-up posts by Nathaniel Good.
permalink: /technical-writing/project-writeups/
category_filter: project-writeups
---

<section class="section">
  <div class="card">
    <ul class="list">
      {% assign filtered = site.posts | where_exp: "post", "post.categories contains page.category_filter" %}
      {% for post in filtered %}
        <li>
          <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
          <span class="item-meta">({{ post.date | date: "%Y-%m-%d" }})</span>
        </li>
      {% endfor %}
      {% if filtered.size == 0 %}
        <li>No posts yet.</li>
      {% endif %}
    </ul>
  </div>
</section>
