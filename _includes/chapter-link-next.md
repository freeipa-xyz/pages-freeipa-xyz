<!-- NEXT CHAPTER BUTTON -->
{% assign theme_chapters = site.articles | where: "layout", page.layout | where: "theme", page.theme | sort: "chapter_id" %}

{% assign next_chapter = nil %}

{% for chapter in theme_chapters %}
    {% if chapter.chapter_id > page.chapter_id %}
        {% if next_chapter %}
            {% if next_chapter.chapter_id > chapter.chapter_id %}
                {% assign next_chapter = chapter %}
            {% endif %}    
        {% else %}
            {% assign next_chapter = chapter %}
        {% endif %}
    {% endif %}
{% endfor %}

{% if next_chapter %}
    <a href={{next_chapter.url}}>[{{next_chapter.chapter_title | escape}} 🡲 ]</a>
{% else %}
    <!-- NO NEXT CHAPTER -->
{% endif %}
<!-- /NEXT CHAPTER BUTTON -->