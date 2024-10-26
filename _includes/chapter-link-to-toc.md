
{% assign theme_page = site.articles | where: "layout", "theme" | where: "theme", page.theme | first %}
<a href="{{ theme_page.url }}">[ ⮤ Theme ToC ]</a>
