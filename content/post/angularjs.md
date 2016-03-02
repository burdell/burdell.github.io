---
title: ListenFirst - Angular.js & Last.fm 
author: nathan griffin
date: 2013-11-18
template: singlepage.html
---

We recently started using [Angular.js](http://angularjs.org/) at work for front-end development after using [Backbone.js](http://backbonejs.org/) for a little over a year. While we were still deciding on whether to to use it or not I wrote a <a target="_blank" href="http://www.nathaniscool.com/ListenFirst">small app</a> to play around with Angular that uses the Last.fm API and tells you the first time you listened to an artist, the last time you've listened, and how many times you've listened to them. (If you don't have a Last.fm account, mine is 'burdell'. Have fun.)

So next time you're with your friends and they ask "Hey, have you heard of this band?". You're response will be ["Uh, yeah. I've listened to them since January 9, 2008"](http://bitchinlifestyle.tv/wp-content/uploads/2013/03/homens-e1347382625816.jpg).

There's sometimes some weird issues with the API sometimes, but not sure if I'm going to go back and make it more robust or not. I've thought about doing something with [D3](http://d3js.org/) and listening data, but not sure what I'd do. So maybe I'll revist it in the future.

Quick thoughts...

### Backbone vs. Angular :: Library vs. Framework ###
Backbone is more library-like. It gives you some (really good) tools and lets you do what you want with them. Angular is more of a framework. It gives you more of a system and puts more restrictions on how you work. 

Both approaches have their advantages, but what ultimately led to switching at work was the leverage provided by Angular's built in stuff, especially the programatic templating in Angular. Backbone views quickly become *monstrous* with all the event handlers and having to program interaction away from the actual DOM.

### Directives? Services? Factories? OH MY! ###
Backbone felt a bunch like OO sometimes with the ability to extend models/views/collections, which I really liked. It was really easy to "get" what it was doing. Angular at first felt like a bunch of disconnected pieces. Once you figured out how these pieces fit together, it made sense and was awesome, but the learning curve to figuring that out was more than with Backbone.

### Last.fm ###
I've thought about building this for awhile and I've used Last.fm since 2006 or so (although I reset my listening history in 2008 :('. The API was pretty easy to use. There were some quirks, but overall I approved. Like I said, I think using D3 to do some kind of visualization would be pretty awesome, but not sure what I'd do with it just yet.

### Backbone? Angular? Why? ###
Backbone and Angular bring MVC-like structure to the front end of your web applications. MVC is a common software architecture pattern in which you seperate out your application into 3 parts:

1. Model: the underlying data in your application
2. View: the presentation of your data
3. Controller: the "business logic" of how your data gets transformed

In a tradition web application, models are whatever database you've chosen, views are your HTML/CSS/JavaScript, and the controller is whatever server-side framework you've chosen (Rails, Django, Node, .Net MVC, etc). 

JavaScript frameworks live entirely in the **V** in MVC, and split that up into its own little MVC-like playground. The advantages for doing this quickly become apparent when trying to do anything more than basic interaction on a web page. HTML/CSS is all about presentation, not the data it's presenting. By adding structure and splitting up concerns, interaction becomes much easier and has given way to the immersive [single page webapps](http://en.wikipedia.org/wiki/Single-page_application) of today.
