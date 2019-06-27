---
layout: post
title:      "Rails Agro Project"
date:       2019-06-27 11:56:14 -0400
permalink:  rails_agro_project
---

## About:

I built this app to help with tracking gardens and plants at the very basic level. Users can create many garden and populate them with plant entries containing details and dates specific to that plant entry.

## Steps:
### 1. Configure gems and dependencies.
```

#Added Gems---

gem 'devise'
gem 'pry'
gem 'omniauth-github'

```

### 2. Set up database tables and fields for app models (Users, Gardens, Plants, Comments).

### 3. Devise for user login features, omniauth, and user authentication.

### 4. Join table migrations for user_gardens, gardens_plants, and comments.

### 5. User model associations (has_many, belongs_to, and has_many...through)

### 5. RESTful CRUD routes

### 6. Controllers, views, and forms

### 7. Refactoring to use partials in forms

### 8. Validation

## Recap:
* In hindsight, I should have included validations earlier in the project in order to avoid simple logic errors within the controllers. I found myself getting errors when checking the views on test records I created with empty values that wouldn't have made it into the database if I had implemented validation early.
