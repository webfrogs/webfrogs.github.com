---
layout: page
title: 欢迎来到我的博客
tagline: Supporting tagline
---
{% include JB/setup %}


---
##关于本博


本博刚刚新建，内容嘛，暂时还没有。^_^

博客是基于jekyll和github:pages创建的，如果你对创建过程感兴趣，[请猛戳这里!](http://jekyllbootstrap.com/)

##文章列表

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

##联系我？
新浪微博：[思圃](http://weibo.com/u/1713195262)

github：[webfrogs](https://github.com/webfrogs)



