<table>
<tr><th>Title</th><th>Date Published</th></tr>
{% for post in site.posts %}
<tr><td><a href="{{ post.url}}">{{ post.title }}</a></td><td>{{ post.date | date: "%Y-%m-%d"}}</td>
{% endfor %}
