---
layout: page
title: Projects
permalink: /projects/
---

<div class="centro">
  <div class="home">

    <div class="row">
      {% for project in site.projects %}
        <div class="col-lg-3 col-lg-3">
            <div class="hovereffect">
                <img class="img-responsive" width="500px" src="{{ project.thumbnail-path }}" alt="{{ alt }}">
              <div class="overlay">
                  <a class="info" href="{{ project.url | prepend: site.baseurl }}"></a>
              </div>
            </div>
            <h4><center>{{ project.title }}</center></h4>
        </div>
      {% endfor %}
    </div>
  </div>
</div>
