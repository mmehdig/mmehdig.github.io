---
layout: page
title: Archive
permalink: /archive/
---

{% for post in site.posts %}
#### [{{ post.title | escape }}]({{ post.url | prepend: site.baseurl }})
##### {{ post.date | date: "%b %-d, %Y" }} #####

{{ post.excerpt }}

{% endfor %}
