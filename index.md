---
layout: home
---

# Welcome to Diedrichsen Lab Wiki

Comprehensive guide to lab procedures, protocols, and resources.

## Guides

{% for guide in site.guides %}
- [{{ guide.title }}]({{ guide.url }})
{% endfor %}

## Tutorials

{% for tutorial in site.tutorials %}
- [{{ tutorial.title }}]({{ tutorial.url }})
{% endfor %}
