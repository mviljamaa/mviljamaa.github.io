---
layout: page
type: section
status: publish
published: true
title: Commercial Software
date: '2011-05-07 10:45:46 +0200'
date_gmt: '2011-05-07 10:45:46 +0200'
categories: []
tags: []
---

I have contributed to closed source projects. Below you will find products etc.
that I have produced code for.

{% assign sorted_pages = site.pages | sort:"order" %}
{% for p in sorted_pages %}
   {% assign splt = p.url | split: page.url %}
   {% if splt.size == 2 and splt[0] == '' %}
      {% assign slash = splt[1] | split: '/' %}
{% if slash.size == 1 %}      
- <a class="page-link" href="{{p.url | prepend: site.baseurl}}">{{p.title}}</a>
{% else %}
   - <a class="page-link" href="{{p.url | prepend: site.baseurl}}">{{p.title}}</a>
{% endif %}
   {% endif %}
{% endfor %}



<!-- <ul class="tuto">

{% assign sorted_pages = site.pages | sort:"order" %}
{% for p in sorted_pages %}
  {% assign splt = p.url | split: page.url %}
  {% if splt.size == 2 and splt[0] == '' %}
     {% assign slash = splt[1] | split: '/' %}
     	<li>
	<a class="page-link" href="{{p.url | prepend: site.baseurl}}">{{p.title}}</a>
	</li>
  {% endif %}
{% endfor %}

</ul> -->

