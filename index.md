---
layout: default
---

<!-- Just some nice to have styles for the pager buttons -->
<style>
  ul.pager { text-align: center; list-style: none; }
  ul.pager li {display: inline;border: 1px solid black; padding: 10px; margin: 5px;}
</style>

<div class="posts">
<!-- This loops through the paginated posts -->
{% for post in paginator.posts %}
  <article class="post">

    <h1><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h1>

    <div class="entry">
      {{ post.excerpt }}
    </div>

    <a href="{{ site.baseurl }}{{ post.url }}" class="read-more">Read More</a>
  </article>
{% endfor %}
</div>

{% if paginator.total_pages > 1 %}
<ul class="pager">
    {% if paginator.previous_page %}
    <li class="previous">
        <a href="{{ paginator.previous_page_path | prepend: site.baseurl | replace: '//', '/' }}">&larr; Newer Posts</a>
    </li>
    {% endif %}
    {% if paginator.next_page %}
    <li class="next">
        <a href="{{ paginator.next_page_path | prepend: site.baseurl | replace: '//', '/' }}">Older Posts &rarr;</a>
    </li>
    {% endif %}
</ul>
{% endif %}