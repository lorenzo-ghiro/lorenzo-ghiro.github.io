---
title: "News"
layout: gridlay
sitemap: false
permalink: /news/
---

## News

<div class="section-card" markdown="0">
<div class="news-timeline">
{% assign news_items = site.news | sort: 'path' | reverse %}
{% if news_items and news_items.size > 0 %}
{% for article in news_items %}
{% include news-item.html news=article show_body=true %}
{% endfor %}
{% else %}
<p class="text-muted" style="margin: 0;">No news entries yet.</p>
{% endif %}
</div>
</div>
