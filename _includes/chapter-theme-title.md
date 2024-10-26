{% assign theme_page = site.articles | where: "layout", "theme" | where: "theme", page.theme | first %}
{{ theme_page.title | escape }}