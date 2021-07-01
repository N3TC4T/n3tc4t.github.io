---
layout: default
---

# $ cat about.txt
{:id="about"}

I love coding and learning new things, in the past years I played with alot of
programming languages, frameworks and platforms and can say I'm a
jack-of-all-trades.

Right now my focus is on building stuff on top of XRP Ledger and Interledger Protocol.

In the end I want to confirm that I can code in HTML too (just kidding :D) 

# $ cat contact.txt
{:id="contact"}

E-mail:

> netcat.av[at]gmail[dot]com

GitHub:

> [https://github.com/n3tc4t](https://github.com/n3tc4t)

Twitter:

> [https://twitter.com/baltazar223](https://twitter.com/baltazar223)

# $ cat posts.txt
{:id="posts"}

<ul>
{% for post in site.categories.posts %}

{% if post.en %}
<li>{{ post.title }} :: <a href="{{ post.url }}" title="{{ post.description }}">en</a></li>
{% endif %}

{% endfor %}
</ul>

# $ cat projects.txt
{:id="projects"}

<ul>
{% for project in site.categories.projects %}
<li><a href="{{ project.link }}">{{ project.title }}</a> - {{ project.description }}</li>
{% endfor %}
</ul>

# $ cat tools.txt
{:id="tools"}

<ul>
{% for tool in site.categories.tools %}
<li><a href="{{ tool.link }}">{{ tool.title }}</a> - {{ tool.description }}</li>
{% endfor %}
</ul>
