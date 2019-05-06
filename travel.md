---
layout: default
map: true
---
# Travel

Some of the places I've been.

<div id="mapid" class="cell -12of12" height="500px"></div>

<script>
    var mymap = L.map('mapid').setView([1, 1], 1);
    L.tileLayer('https://tile.openstreetmap.org//{z}/{x}/{y}.png', {
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>',
        maxZoom: 18,
    }).addTo(mymap);

    {% for place in site.travel %}
    var marker_{{place.name}} = L.marker([{{place.latitude}}, {{place.longitude}}]).addTo(mymap);
    marker_{{place.name}}.bindPopup('{{place.title}}');
    {% endfor %}
</script>
