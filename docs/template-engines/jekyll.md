---
layout: default
title: Jekyll
parent: Template Engines
nav_order: 10
---

There are two primary document collections in Jekyll: posts and pages. Another will be collections if you use them.

Pages and posts may contain their own front matter, so I suggest you treat each one separately.

### List of all posts
Read the Jekyll array containing a list of all the posts:
```jekyll
{% assign posts = site.posts %}
{% for post in posts %}
    title: {{post.title}}
{% endfor %}
```

### List of all pages
Read the Jekyll array containing a list of all the pages:

```jekyll
{% assign pages = site.pages | where_exp: 'page', 'page.title' %}
{% for page in pages %}
    title: {{page.title}}
{% endfor %}
```

Notice I used a where_exp filter in the list of pages. Sometimes a page does not have a title in the front matter. That will remove untitled pages from the listing. If you want all of them, even without a title, then the code would look like this instead:

```jekyll
{% assign pages = site.pages %}
{% for page in pages %}
    title: {{page.title}}
{% endfor %}
```

### Site Variables
[Site Variables](https://jekyllrb.com/docs/variables/#site-variables) show the list of built-in Jekyll site variables.
You can learn about the different arrays you can access that Jekyll automatically collects for you. There may be other document types (like HTML docs) that you want to include as well.