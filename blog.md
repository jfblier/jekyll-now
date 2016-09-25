---
layout: page
title: Blog
permalink: /blog/
---

<div class="posts">
  {% for post in site.posts %}
    <article class="post">
      <h2><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h2>
	  <div class="date">
		Posted on <a href="{{ site.baseurl }}{{ post.url }}">{{ page.date | date: "%B %e, %Y" }}</a>
	  </div>
	  {% if page.image %}
		  <div class="image">
			<img src="{{ site.url }}/images/{{ page.image }}"/>
		  </div>
	  {% endif %}

      <div class="entry">
        {{ post.excerpt }}
      </div>

      <a href="{{ site.baseurl }}{{ post.url }}" class="read-more">Read More</a>
    </article>
  {% endfor %}
</div>