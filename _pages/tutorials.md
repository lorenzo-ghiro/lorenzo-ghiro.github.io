---
title: "Tutorials"
layout: gridlay
sitemap: true
permalink: /tutorials/
---

## Tutorials

Hands-on exercises written for students in my courses on vehicular networking and simulation.

<div class="tutorials-grid" markdown="0">
{% assign sorted_tutorials = site.tutorials | sort: "title" %}
{% for tut in sorted_tutorials %}
<div class="section-card tutorial-card">
  <h4><a href="{{ tut.url | prepend: site.baseurl }}">{{ tut.title }}</a></h4>
  {% if tut.tags %}
  <div class="skill-chips">
    {% for tag in tut.tags %}<span class="skill-chip">{{ tag }}</span>{% endfor %}
  </div>
  {% endif %}
  <p>{{ tut.description }}</p>
  <a href="{{ tut.url | prepend: site.baseurl }}" class="btn-pill btn-website">Read tutorial &rarr;</a>
</div>
{% endfor %}
</div>
