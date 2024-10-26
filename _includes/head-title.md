{% assign theme_page = site.articles | where: "layout", "theme" | where: "theme", page.theme | first %}
{% if theme_page.title %}
  {% if page.chapter_title %}
    {% capture head_title %}
    {{ theme_page.title }}: {{ page.chapter_title }} | {{ site.title }}
    {% endcapture %}
  {% endif %}
{% endif %}




