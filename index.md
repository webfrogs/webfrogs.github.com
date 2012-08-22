---
layout: page
title: 欢迎来到我的博客
tagline: Supporting tagline
---
{% include JB/setup %}


---
##关于本博


博客是基于jekyll和github:pages创建的，如果你对创建过程感兴趣，[请猛戳这里!](http://jekyllbootstrap.com/)

搭建这样一个属于自己的个人博客，是一个愉快的过程。在这个过程中，感受到了许多技术上的闪光点。以后的日子里，我会逐渐用这个博客来记录一些事情。

##文章列表

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

##联系我？
新浪微博：[思圃](http://weibo.com/u/1713195262)

github：[webfrogs](https://github.com/webfrogs)



