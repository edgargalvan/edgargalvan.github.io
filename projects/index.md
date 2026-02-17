---
layout: page
title: Projects
permalink: /projects/
---

{% assign sorted = site.projects | sort: "order" %}
{% for p in sorted %}
- [{{ p.title }}](/projects/{{ p.slug }}/) — {{ p.summary }}
{% endfor %}
