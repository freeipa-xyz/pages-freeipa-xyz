---
layout: default
---

<article class="post">

  <header class="post-header">
    <h2 style="margin:0;  color: var(--minima-text-color); filter: brightness(50%);">{%- include chapter-theme-title.md -%}</h2>
    <h1 class="post-title">{{ page.chapter_title | escape }}</h1>
  </header>
  <hr />
  <div class="post-content">
    {{ content }}
  </div>

</article>

<div style="display: flex; flex-flow: row nowrap; justify-content: space-between;">
<span style="filter: brightness(70%);">{%- include chapter-link-to-toc.md -%}</span>
<span><span style="filter: brightness(70%); margin-right: 20px;">{%- include chapter-link-prev.md -%}</span> {%- include chapter-link-next.md -%}</span>
</div>

