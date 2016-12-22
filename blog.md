---
layout: default
---
# Posts

{% for post in site.posts%}
[{{ post.date | date: "%F" }}] [**{{ post.title }}**]({{post.url}})
{% endfor %}
