---
layout: page
---

    {% for post in site.posts %}
<div class="card">
    <div class="card-header">
        <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    </div>
    <div class="card-content">
        <h3>{{ post.excerpt }}</h3>
        <h4><a href="{{ post.url }}">Read more</a></h4>
    </div>
</div>
  {% endfor %}

<!-- <div class="card">
    <div class="card-header">
        <h2>A pragmatic developer working on high-quality code and evolutionary architectures.</h2>
    </div>
    <div class="card-content">
        <h3>Access my articles by clicking on the icons below</h3>
        <a href="https://medium.com/@allansduarte">
            <i class="fab fa-medium" title="allansduarte's Medium Profile"></i>
        </a>
        <a href="https://dev.to/allansduarte">
            <i class="fab fa-dev" title="allansduarte's DEV Profile"></i>
        </a>
    </div>
</div>

<div class="card">
    <div class="card-header">
        <h2>Elixir's mentor at Exercism</h2>
    </div>
    <div class="card-content">
        <a href="https://exercism.io/profiles/allansduarte">
            <img src="https://assets.exercism.io/social/general.png" alt="Exercism Elixir Mentor" />
        </a>
        <a href="https://exercism.io/profiles/allansduarte">
            <img src="https://assets.exercism.io/tracks/elixir-bordered-turquoise.png" alt="Exercism Elixir Mentor" />
        </a>
    </div>
</div>

<div class="card">
    <div class="card-header">
        <h2>CodersRank Profile</h2>
    </div>
    <div class="card-content">
        <codersrank-widget username="allansduarte"></codersrank-widget>
    </div>
</div> -->
