---
layout: page
title: "Tai Chi & Bagua Practice Hub"
permalink: /tai-chi/
description: "Student reference guide for traditional Tai Chi form and Bagua Palm Changes."
---
{::options parse_block_html="true" /}

<div class="somatic-hero taichi-theme">
<p>
Welcome! This hub is designed for our <strong>Tai Chi</strong> and <strong>Bagua Circle Walking</strong> students. It contains structural alignment cues, circle walking mechanics, and references for the Dragon Palm Changes.
</p>
<div class="mobile-tip">
📱 <strong>Student Tip:</strong> Save this page for quick access to your training sequences while practicing at home!
</div>
</div>

---

## 🏛️ Foundations: The Structural Stack (Wu Chi)

Every Tai Chi and Bagua movement is built upon the **Wu Chi** posture. Practice standing in Wu Chi for 2–5 minutes before any sequence:

1. **The Plantar Base:** Position feet hip-width and parallel. Sense the weight distributed across the **9 Yang points** of each foot (toes, balls, edges, and heel).
2. **The Soft Joint Stack:** Micro-bend the knees. Let the sacrum (tailbone) sink straight down to flatten the lumbar spine.
3. **The Suspended Crown:** Tuck your chin slightly and press the fontanelle (back-third of your skull) up toward the sky as if suspended by a string.
4. **The Relaxed Core:** Relax your sternum down into your hips. Keep your chest hollow and shoulders dropped.

---

## 🌀 Bagua Circle Walking

Circle Walking is the core training method of Bagua. It trains continuous balance, spiral fascia lines, and directional changes:

* **The Stance:** Walk around the edge of an imaginary circle. Turn your waist and torso slightly toward the center of the circle.
* **The Step (Mud Step):** Slide your feet forward close to the floor without rising up or sinking down.
* **The Arms (Dragon Palms):** Extend your arms into "Dragon Palms" or "Single Palm" shapes, focusing your gaze through the fingers of your leading hand toward the center of the circle.
* **Timing:** Walk 8–10 rotations in one direction, perform a **Palm Change** to turn the wheel, and walk the opposite direction.

---

## 🐉 Master Wei Lun Huang's Dragon Palm Changes

These traditional Bagua Palm Changes are used to transition and change directions while walking the circle. Click the linked posts to read full descriptions of each sequence:

1. **🔗 [Single Palm Change]({% post_url 2025-10-25-MasterWeiLunHaungDragonPalmChanges %}):** Cut and clear the pie, Carry the wood, Pull up the shutter, Hide the flower under the leaf, Lonely goose flies out, Pair of geese fly together.
2. **Double Palm Change:** Includes twisting the ball of the inside foot, spear-hand extension, and turning back to dragon palms.
3. **Same Step Palm Change (Bullwinkle):** Direct step-through spear hand strike.
4. **Reverse Step Palm Change:** Twist and strike to the chin, kick groin, throw off helmet.
5. **Twisting Body:** immediate spear hand and body throw.
6. **Threading Through:** Symmetrical grab, pull, and triple spear-hands.

---

## 📖 Deep-Dive Blog Resources

Here are some of my articles diving deeper into the science and mechanics behind Tai Chi, Bagua, and internal power cultivation:

<div class="recent-posts-grid">
{% for post in site.posts %}
{% if post.tags contains 'taichi' or post.tags contains 'bagua' %}
<div class="post-card">
<span class="post-card-date">{{ post.date | date: "%B %d, %Y" }}</span>
<h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
<p>{{ post.description }}</p>
</div>
{% endif %}
{% endfor %}
</div>
