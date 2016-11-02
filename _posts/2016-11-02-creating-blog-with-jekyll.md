---
layout: post
title: Creating a blog with Jekyll
---

After years of using wordpress I found [Jekyll](https://jekyllrb.com/) and [Github pages](https://pages.github.com/) and I think it is awesome. 

Jekyll is a static page generator written in Ruby. I have always used wordpress for blogs before. Wordpres is a great tool, but I am a programmer, I like to write in Vim, use [markdown](http://markdowntutorial.com/) and git. And Jekyll with Github pages allows me just that. This blog post is written in markdown and [versioned in git repository](https://github.com/DominikMostek/dominikmostek.github.io/tree/master/_posts).

Jekyll support plugins but be aware that only few of them [can be used](https://help.github.com/articles/configuring-jekyll-plugins/) when hosting on github pages.

Creating jekyll site is very easy but even easier is to use already existing template projects like [Beautiful Jekyll](http://deanattali.com/beautiful-jekyll/) which I am using. It is very customizable and easy to use. 

Running Jekyll localy cannot be easier 

~~~
~/dominikmostek.github.io$ bundle exec jekyll server
~~~
and your site is up and running.

[Drafts](https://jekyllrb.com/docs/drafts/) are also supported. 

Runnig a simple blog cannot be easier.