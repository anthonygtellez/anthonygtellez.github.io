---
layout: articles
title: My Confrence Presentations
show_title: false
articles:
  data_source: site.posts
  article_type: BlogPosting
  show_cover: false
  show_excerpt: true
  show_readmore: true
  show_info: true

---

{{ content }}
<div class="layout--home">
  {%- include paginator.html -%}
</div>
<script>
  {%- include scripts/home.js -%}
</script>

{{ content }}