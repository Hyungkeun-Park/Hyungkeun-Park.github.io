---
title: "NLP"
layout: archive
permalink: categories/NLP
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.NLP %}  <!--Change category-->
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}

<!-- add to /_includes/nav_list_main -->
