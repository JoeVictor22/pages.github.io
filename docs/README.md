# Landing page

you landed here :3

## hello

hello there

## posts?

<ul class="posts">
{% for post in site.posts %}
  <div class="post_info">
    <li>
         <a href="{{ post.url }}">{{ post.title }}</a>
         <span>({{ post.date | date:"%Y-%m-%d" }})</span>
    </li>
    </div>
  {% endfor %}
</ul>


## bye

hasta la vista
