---
title: Post 0 - Now with Wintersmith!
author: nathan griffin
date: 2013-09-17
---

Officially moved nathaniscool.com to using a new static site generator called [Wintersmith](http://www.wintersmith.io) and hosting through GitHub pages. It was really easy to get Wintersmith up and running and it runs on Node which I've kinda been interested in for while. Annnd it means I can use Handlebars and Underscore, both at which I've used for awhile and really, really like.

### ... what is a "static site generator"?
A static site generator takes a bunch of files on a computer, works some magic, and spits out a full site written in only HTML, CSS, and JavaScript (the love languages of the browser). 

### But, why?
1. It's cool
2. Since the entire site is HTML, CSS, and JavaScript, the options for hosting it increase exponentially and moving hosting solutions is super, super easy. If I wanted to move to something like Amazon S3, or even set up my own server, all I have to do is move the files.
3. You can't [hax0r](http://cdn-www.i-am-bored.com/media/95505_rushack2.jpg) HTML. There's no login to an account to make a post, and no database connection to retrieve the content from.
4. It allows you to really seperate out different parts of your site when you're building it. For example, the sidebar, header, and main content panel of this site are all in seperate files. If I needed to make a change to one of those, it's super easy to find it and make changes that then apply to every page that uses those sections. Every post is a seperate markdown file and contains meta-data about that post. But then you just run a command line a tool and build everything, and boom, you've got a site.
5. It's cool

### Why Wintersmith?
I looked at [Jekyll](http://jekyllrb.com/) and actually made a site with [Octopress](http://octopress.org/), which is written on top of Jekyll. Octopress is probably the most widely used static site generator, so it has a bunch of really cool themes that people have written for it. But 1. My Ruby chops are minimal and 2. it felt really hard to understand how stuff was put together. 

When I started using Wintersmith, I felt like it just got out of my way and it made a bunch of sense in how you set up and get everything up and running. 

And it's cool.


  



