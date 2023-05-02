---
layout: page
title: Projects
permalink: /projects/
---
## [monetholt/quaver-n-bits](https://github.com/monetholt/quaver-n-bits)

Musician Finder App (Collaborative Node.js / MySQL / CSS App)

Language: CSS

Last Updated: 13 Aug 2020

{% for repo in site.github.public_repositories %}

{% if repo.fork == false %}

## [{{ repo.name }}]({{ repo.html_url }})

{{ repo.description }}

Language: {{ repo.language }}

Last updated: {{ repo.pushed_at | date_to_string }}

{% endif %}

{% endfor %}