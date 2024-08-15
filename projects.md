---
layout: page
title: Projects
permalink: /projects/
---


<ul>
  {% for project in site.projects %}
    <li>
      <h2><a href="{{ project.project_url }}">{{ project.title }}</a></h2>
      <p>{{ project.description }}</p>
      <p><a href="{{ project.github_url }}">GitHub Repository</a></p>
    </li>
  {% endfor %}
</ul>
