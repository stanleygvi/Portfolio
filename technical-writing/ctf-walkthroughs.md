---
layout: technical-page
title: CTF Walkthroughs
subtitle: CTF challenge documentation from recon to solve path.
description: CTF walkthrough posts by Nathaniel Good.
permalink: /technical-writing/ctf-walkthroughs/
category_filter: ctf-walkthroughs
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
