---
layout: default
---
# Projects

{% for project in site.projects %}
[**{{ project.title }}**]({{ site.baseurl }}{{project.url}})
{% endfor %}
