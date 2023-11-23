---
layout: page
permalink: /collaborators/
title: collaborators
description:
nav: true
nav_order: 4
---

TODO list
- Pick a 2-color complementary color scheme? (maybe)
- Host the website on satyaki.net and not github.io
- add teaching resouces
- Fix google analytics
- Remove repeated entries for abstracts. Add them as awards.. Change bib.html to include award 
"""{% if entry.award %}
      <!-- <span class="title"> <strong class="award">&lt; {{entry.award}}&gt; </strong> {{entry.title}}</span> -->
      <span class="title"> <strong class="award"> &#x272D;{{entry.award}}&#x272D; </strong> {{entry.title}}</span>
    {% else %}
      <span class="title">{{entry.title}}</span>
    {% endif %}
"""