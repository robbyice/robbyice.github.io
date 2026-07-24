---
layout: single
title: Games
permalink: /games/
author_profile: true
---

<div class="portfolio-page" markdown="1">

{% assign games = site.games | sort: "order" %}
{% for game in games %}
{% include game-detail.html game=game %}
{% endfor %}

</div>
