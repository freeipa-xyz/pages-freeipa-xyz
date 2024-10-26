<!-- PREV CHAPTER BUTTON -->
{% assign theme_chapters = site.articles | where: "layout", page.layout | where: "theme", page.theme | sort: "chapter_id" %}

{% assign prev_chapter = nil %}

{% for chapter in theme_chapters %}
    {% if chapter.chapter_id < page.chapter_id %}
        {% if prev_chapter %}
            {% if prev_chapter.chapter_id < chapter.chapter_id %}
                {% assign prev_chapter = chapter %}
            {% endif %}    
        {% else %}
            {% assign prev_chapter = chapter %}
        {% endif %}
    {% endif %}
{% endfor %}

{% if prev_chapter %}
    <a href={{prev_chapter.url}}>[ 🡰 {{prev_chapter.chapter_title | escape}} ]</a>
{% else %}
    <!-- NO PREV CHAPTER -->
{% endif %}
<!-- /PREV CHAPTER BUTTON -->