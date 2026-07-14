---
layout: default
title: Accueil
permalink: /
description: Portfolio professionnel d'Eric Ginez, Data Engineer.
---

<section class="hero">
  <div class="container hero-grid">
    <div class="hero-content">
      <p class="eyebrow">
        Portfolio professionnel · Data Engineering
      </p>

      <h1>
        Je conçois des systèmes de données fiables,
        automatisés et exploitables.
      </h1>

      <p class="hero-description">
        Data Engineer spécialisé dans la conception de pipelines,
        l’orchestration des flux, les architectures cloud, la qualité
        des données et les systèmes d’intelligence artificielle.
      </p>

      <div class="hero-actions">
        <a
          class="button button-primary"
          href="{{ '/projets/' | relative_url }}"
        >
          Découvrir mes projets
        </a>

        <a
          class="button button-secondary"
          href="https://github.com/{{ site.author.github }}"
          target="_blank"
          rel="noopener noreferrer"
        >
          Consulter mon GitHub
        </a>
      </div>

      <ul class="hero-metrics" aria-label="Présentation synthétique">
        <li>
          <strong>12</strong>
          <span>projets techniques</span>
        </li>

        <li>
          <strong>603 h</strong>
          <span>de projets encadrés</span>
        </li>

        <li>
          <strong>Niveau 7</strong>
          <span>certification RNCP</span>
        </li>
      </ul>
    </div>

    <aside class="hero-panel" aria-label="Domaines de compétences">
      <div class="hero-panel-header">
        <span class="status-dot" aria-hidden="true"></span>
        <span>Domaines de spécialisation</span>
      </div>

      <div class="specialization-list">
        <article>
          <span class="specialization-number">01</span>

          <div>
            <h2>Pipelines de données</h2>
            <p>
              Collecte, transformation, validation et exposition
              de données fiables.
            </p>
          </div>
        </article>

        <article>
          <span class="specialization-number">02</span>

          <div>
            <h2>Cloud et orchestration</h2>
            <p>
              Industrialisation des traitements avec Docker,
              Kestra, Airbyte et AWS.
            </p>
          </div>
        </article>

        <article>
          <span class="specialization-number">03</span>

          <div>
            <h2>Data et intelligence artificielle</h2>
            <p>
              Machine Learning, systèmes RAG, bases vectorielles
              et exploitation métier.
            </p>
          </div>
        </article>
      </div>
    </aside>
  </div>
</section>

<section class="home-section">
  <div class="container">
    <div class="section-heading">
      <p class="eyebrow">Expertise technique</p>

      <h2>De la donnée brute jusqu’à son exploitation</h2>

      <p>
        Mon parcours couvre les principales étapes d’un projet Data :
        modélisation, ingestion, transformation, stockage, orchestration,
        tests, déploiement et restitution.
      </p>
    </div>

    <div class="expertise-grid">
      <article class="expertise-card">
        <span class="card-index">01</span>
        <h3>Data Engineering</h3>

        <p>
          Conception de pipelines ELT, architectures Bronze–Silver–Gold,
          traitement batch et streaming.
        </p>

        <ul class="tag-list">
          <li>Python</li>
          <li>SQL</li>
          <li>PostgreSQL</li>
          <li>Spark</li>
          <li>Kafka</li>
        </ul>
      </article>

      <article class="expertise-card">
        <span class="card-index">02</span>
        <h3>Infrastructure et Cloud</h3>

        <p>
          Conteneurisation, orchestration, automatisation et déploiement
          d’infrastructures de données.
        </p>

        <ul class="tag-list">
          <li>Docker</li>
          <li>Kestra</li>
          <li>Airbyte</li>
          <li>AWS</li>
          <li>GitHub</li>
        </ul>
      </article>

      <article class="expertise-card">
        <span class="card-index">03</span>
        <h3>Data Science et IA</h3>

        <p>
          Préparation des données, modèles prédictifs, API et systèmes
          de recherche augmentée.
        </p>

        <ul class="tag-list">
          <li>Scikit-learn</li>
          <li>FastAPI</li>
          <li>LangChain</li>
          <li>FAISS</li>
          <li>RAG</li>
        </ul>
      </article>
    </div>
  </div>
</section>