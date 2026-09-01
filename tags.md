---
layout: page
title: Tags
permalink: /tags/
---

{%- assign tags = site.tags | sort -%}
{%- if tags.size > 0 -%}
<p class="post-tags">
{%- for t in tags -%}
  <a class="post-tag" href="{{ t[0] | slugify | prepend: '/' | append: '/' | relative_url }}">{{ t[0] }} ({{ t[1] | size }})</a>
{%- endfor -%}
</p>

{%- assign date_format = site.minima.date_format | default: "%Y-%m-%d" -%}
{%- for t in tags %}
<h2 id="{{ t[0] | slugify }}">{{ t[0] }}</h2>
<ul class="post-list-tight">
{%- for post in t[1] %}
  <li><span class="post-meta">{{ post.date | date: date_format }}</span> <a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a></li>
{%- endfor %}
</ul>
{%- endfor -%}
{%- else -%}
<p>No tags yet.</p>
{%- endif -%}
