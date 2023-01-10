# Landing page

you landed here

:3

## hello

hello there

## posts?

<ul>
  {% assign posts = site.posts | where:"static", "true" %}

  {% for post in posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>


## bye

hasta la vista
