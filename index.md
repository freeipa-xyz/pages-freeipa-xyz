---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---

<ul>
{% for article in site.articles %}
  {% if article.layout == "theme" %}
  <li><a href="{{ article.url }}">{{ article.title | escape }}</a></li>
  {% endif %}
{% endfor %}
</ul>
