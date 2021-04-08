---
layout: archive
title: "Sitemap"
permalink: /sitemap/
author_profile: true
---

{% include base_path %}

A list of all the posts and pages found on the site. For you robots out there is an [XML version]({{ base_path }}/sitemap.xml) available for digesting as well.


<h2>Blog Posts</h2>
{% for post in site.posts %}
  {% include archive-single.html %}
{% endfor %}

<h2>Essay</h2>
{% for post in site.essay %}
  {% include archive-single.html %}
{% endfor %}
