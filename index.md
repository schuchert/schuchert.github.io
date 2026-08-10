---
layout: page
title: Home Portal
permalink: /
---

<div class="welcome-section">
  <p class="welcome-intro">
    Welcome to my central hub. This is where I share my personal writings and curate resources for my students practicing <strong>Qigong</strong>, <strong>Tai Chi</strong>, and the <strong>Science of Stretching (SoS)</strong>.
  </p>
</div>

## 📱 Student Practice Hubs
Select a portal below to access curated sequences, homework guides, and video references for your home practice. These pages are optimized for mobile phones—perfect for scanning via QR codes!

<div class="portal-grid">
  <div class="portal-card">
    <div class="portal-card-icon">🌀</div>
    <h3>Qigong Practice</h3>
    <p>A comprehensive sequence of flows to cultivate balance, structural alignment, and stress reduction.</p>
    <a href="/qigong/" class="btn">Enter Qigong Hub &rarr;</a>
  </div>
  
  <div class="portal-card">
    <div class="portal-card-icon">☯️</div>
    <h3>Tai Chi & Bagua</h3>
    <p>Traditional forms, martial applications, and deep internal work to build somatic coordination.</p>
    <a href="/tai-chi/" class="btn">Enter Tai Chi Hub &rarr;</a>
  </div>
  
  <div class="portal-card">
    <div class="portal-card-icon">🧘‍♂️</div>
    <h3>Science of Stretching</h3>
    <p>Targeted deep flexibility routines focusing on happy joints, hips, shoulders, and long-hold release.</p>
    <a href="/stretching/" class="btn">Enter Stretching Hub &rarr;</a>
  </div>
</div>

---

## 🎥 YouTube Practice Channel
I regularly upload free, short-form instructional videos and full-length sequences on my YouTube channel.

<div class="youtube-banner">
  <p>
    ✨ <strong>Subscribe and practice with me:</strong> 
    <a href="https://www.youtube.com/@brettschuchert" target="_blank" rel="noopener">@brettschuchert on YouTube</a>
  </p>
</div>

---

## ✍️ Recent Writing & Musings
I write about somatic calibration, metabolic health, intentional habits, and personal epiphanies. Here are my latest posts:

<div class="recent-posts-grid">
  {% for post in site.posts limit:3 %}
    <div class="post-card {% if post.draft %}draft{% endif %}">
      <span class="post-card-date">{{ post.date | date: "%B %d, %Y" }}</span>
      <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
      <p>{{ post.description }}</p>
    </div>
  {% endfor %}
</div>

<div style="text-align: center; margin-top: 2.5em; margin-bottom: 2em;">
  <a href="/blog/" class="lbl-toggle" style="display: inline-block; padding: 12px 28px; text-decoration: none;">Explore Full Blog Archive &rarr;</a>
</div>
