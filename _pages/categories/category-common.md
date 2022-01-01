---
title: "Common test"
layout: archive
permalink: categories/common"
author_profile: true
sidebar_main: true
---


{% assign posts = site.categories.Common %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
