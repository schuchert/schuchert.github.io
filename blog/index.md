<table>
<tr><th>Title</th><th>Date Published</th></tr>
{% assign posts_reversed = site.posts | reverse %}
{% for post in posts_reversed %}
<tr><td><a href="{{ post.url}}">{{ post.title }}</a></td><td>{{ post.date | date: "%Y-%m-%d"}}</td>
{% endfor %}
