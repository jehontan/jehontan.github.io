---
layout: default
---
# Travel

A list of some of the places I've been.

{% for place in site.travel%}

<div class="media">
  <div class="media-left">
    <div class="avatarholder" src={{ place.image_preview }}></div>
  </div>
  <div class="media-body">
    <div class="media-heading">{{ place.title }}</div>
    <div class="media-content">{{ place.short_description }}</div>
  </div>
</div>

{% endfor %}
