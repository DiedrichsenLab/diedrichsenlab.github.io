---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---

Welcome to Diedrichsen Lab Wiki. 


## Guides

{% for guide in site.guides %}
- [{{ guide.title }}]({{ guide.url }})
{% endfor %}