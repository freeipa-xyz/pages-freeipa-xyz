
{% assign theme_chapters = site.articles | where: "layout", "chapter" | where: "theme", page.theme | sort: "chapter_id" %}

<ul>
{% for chapter in theme_chapters %}
  {% capture chapter_link_text %}{{ chapter.chapter_title }}{% endcapture %}
  <li><a href="{{chapter.url}}">{{chapter_link_text | escape}}</a></li>
{% endfor %}
</ul>
