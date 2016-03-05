---
layout: page
status: publish
published: true
title: Blog
date: '2011-05-07 10:45:02 +0200'
date_gmt: '2011-05-07 10:45:02 +0200'
categories: [section]
tags: []
---



<ul class="posts">
<h1>Posts/h1>
    {% for post in site.posts %}
    <li>
        <span class="post-date">{{ post.date | date: "%b %-d, %Y" }}</span>
        <a class="post-link" href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
    </li>
    {% endfor %}
</ul> 
