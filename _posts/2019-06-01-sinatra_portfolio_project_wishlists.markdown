---
layout: post
title:      "Sinatra Portfolio Project: Wishlists"
date:       2019-06-01 23:12:44 +0000
permalink:  sinatra_portfolio_project_wishlists
---

https://github.com/darrendaz/sinatra-portfolio-project

Built in Sinatra using ActiveRecord, the aim of this project was to create a simple web app for users to create an authenticated account, public & private wishlists and add items from a shared database.

Originally, I wanted to build an app for tracking ping pong players, matches and sets, and a leaderboard, but that ended up being a bit ambitious. In the end, I found that managing lists would be inherently easier to do with vanilla bootstrap html/css and Sinatra/ActiveRecord. Having to authenticate multiple users in a shared session seemed like uncharted territory and at some point I will be prepared to tackle that challenge, but not for this first project.

Instead, I focused in on just getting my app working and meeting the requirements while commiting to github.

The video below was extremely useful and helped me set up my config.ru, environment.rb, rakefile, and gemfile with very few hiccups. Most of which were careless mistakes on my part.

<iframe width=80% src="https://www.youtube.com/embed/_S1s6R-_wYc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Once I did my `bundle install` and `rake db:create_migration` commands and successfully routing to my index.erb page I was feeling pretty confident.

I found myself getting frustrated in three scenarios:
* mapping and creating routes to erb views
* using a seemingly hacky way to do a patch method with a hidden input
**
* deviating from the engineering of the data with things like interactivity and styling
** I kept having to remind myself that the objective of this project was not to show off my abilities with fancy JS or CSS but rather to show that I know how to connect views with the correct data from the databases that I architected.

