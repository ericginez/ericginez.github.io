---
layout: default
title: Projets
permalink: /projets/
description: Projets Data Engineering réalisés par Eric Ginez.
---

<section class="projects-page">
  <div class="container">
    <header class="projects-heading">
      <p class="eyebrow">
        Réalisations
      </p>

      <h1>Projets Data Engineering</h1>

      <p>
        Ces projets couvrent l’ensemble du cycle de vie des données :
        analyse, modélisation, ingestion, stockage, transformation,
        orchestration, qualité, cloud, visualisation et intelligence
        artificielle.
      </p>
    </header>

    {% assign ordered_projects = site.projects | sort: "order" %}

    <div class="projects-grid">
      {% for project in ordered_projects %}
        {% include project-card.html project=project %}
      {% endfor %}
    </div>
  </div>
</section>