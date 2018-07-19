<table>
<tr>
  <th width="35%">Title</th>
  <th width="15%">Date Published</th>
  <th width="50%">Description</th>
</tr>
{% for post in site.posts %}
<tr {% if post.layout contains "Draft" %}class="draft"{% endif %}>
  <td><a href="{{ post.url}}">{{ post.title }} {% if post.tagline %}<span class="tagline"> - {{ post.tagline }}</span>{% endif %}</a></td>
  <td><a href="{{ post.url}}">{{ post.date | date: "%Y-%m-%d"}}</a></td>
  <td><a href="{{ post.url}}">{{ post.description }}</a></td>
</tr>
{% endfor %}
