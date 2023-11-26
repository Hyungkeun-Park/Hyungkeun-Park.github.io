---
title: "Paper Review"
layout: archive
permalink: categories/Review
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.Review %}  <!--Change category-->
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}

<!-- add to /_includes/nav_list_main -->
