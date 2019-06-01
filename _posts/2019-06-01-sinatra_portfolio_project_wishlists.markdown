---
layout: post
title:      "Sinatra Portfolio Project: Wishlists"
date:       2019-06-01 19:12:45 -0400
permalink:  sinatra_portfolio_project_wishlists
---

https://github.com/darrendaz/sinatra-portfolio-project

Built in Sinatra using ActiveRecord, the aim of this project was to create a simple web app for users to create an authenticated account, public & private wishlists and add items from a shared database.

Originally, I wanted to build an app for tracking ping pong players, matches and sets, and a leaderboard, but that ended up being a bit ambitious. In the end, I found that managing lists would be inherently easier to do with vanilla bootstrap html/css and Sinatra/ActiveRecord. Having to authenticate multiple users in a shared session seemed like uncharted territory and at some point I will be prepared to tackle that challenge, but not for this first project.

Instead, I focused in on just getting my app working and meeting the requirements while commiting to github.

The video below was extremely useful and helped me set up my config.ru, environment.rb, rakefile, and gemfile with very few hiccups. Most of which were careless mistakes on my part.

<iframe width="560" height="315" src="https://www.youtube.com/embed/_S1s6R-_wYc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Once I did my `bundle install` and `rake db:create_migration` commands and successfully routing to my index.erb page I was feeling pretty confident.

I found myself getting the most frustrated in three scenarios:

**mapping and creating routes to erb views**
Thinking through these routes and manually writing them to point at and receive each other was a pretty big pain in the side.  There were a lot of places to go wrong and I found myself constantly jumping in and out of my terminal with `binding.pry` to make sure the params data was getting passed correctly and that the views were not broken. I kept thinking there had to be a better way to abstract it all to at least get the bare bones down and it turns out that Rails helps with this (at least from what I've heard).

**using a seemingly hacky way to do a patch method with a hidden input**
I really did not like the idea of using a hidden input field with the `_method` and `patch` attributes to accomplish the patch method. Same goes for the `delete` method. Both required additional steps as if they were afterthoughts to `GET` and `POST` methods. I felt they both could have been handled more eloquently by Sinatra.

**deviating from the engineering of the data with things like interactivity and styling**
I kept having to remind myself that the objective of this project was not to show off any abilities with fancy JS or CSS but rather to show that I can build an app with connected views using data from the databases I architected. There were many moments where I would find myself wanting to lookup a way to display errors with javascript, handle submitting form inputs with javascript, or making pixel perfect UI. However, if I hadn't reminded myself of accomplishing the minimum viable product (MVP) and stopped myself from rabbit holing as much as I did, I would still be scouring the internet for "how to make a checkbox work within a table using Sinatra".


**WHY DO `:items` HAVE `:quantity`?**
Frankly, I pivoted my project and cut it short for submission. I have plans to build upon the concept of this app in the future as it is relevant e-commerce software. Dealing with the quantity data was something of a challenge and it would have led to numerous changes to all of my models, controllers, and views just to get it barely working. I learned that I need to plan out what I want to do with that data the next time I do a similar project.
