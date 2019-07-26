---
layout: post
title:      "Rails Agro Project + JavaScript"
date:       2019-07-26 02:30:48 +0000
permalink:  rails_agro_project_javascript
---

## About:

This is a continuation of the previous Rails Agro Project but with the addition of javascript files and JSON rendering.

## Steps:
### 1. Configure gems and dependencies.

I knew that I needed to use devise for user signup and login, pry for debugging, and omniauth-github to work with devise. So I added the following to my `Gemfile`

```
#Added Gems---

gem 'devise'
gem 'pry'
gem 'omniauth-github'
```

### 2. Devise for user login features, omniauth, and user authentication.

I followed [this very helpful guide](https://medium.com/@salmaeng71/devise-authentication-guide-with-github-omniauth-for-rails-application-220aa52d5b82) to get devise wired up with my user model and ended up with the below in my `schema.rb` for the users table.

```
  create_table "users", force: :cascade do |t|
    t.string "name", default: "", null: false
    t.string "email", default: "", null: false
    t.string "encrypted_password", default: "", null: false
    t.string "reset_password_token"
    t.datetime "reset_password_sent_at"
    t.datetime "remember_created_at"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.string "provider"
    t.string "uid"
    t.integer "sign_in_count", default: 0
    t.datetime "current_sign_in_at"
    t.string "current_sign_in_ip"
    t.string "last_sign_in_ip"
    t.datetime "last_sign_in_at"
    t.index ["email"], name: "index_users_on_email", unique: true
    t.index ["reset_password_token"], name: "index_users_on_reset_password_token", unique: true
  end
```

### 3. Set up database tables and fields for app models (Users, Gardens, Plants, Comments).

The next thing I needed to do was figure out the rest of my data architecture and relationships. Initially I thought about having just Users, Gardens, and Plants, but decided to add comments to use as a join table with user submittable fields. 

```
  create_table "comments", force: :cascade do |t|
    t.text "contents"
    t.integer "plant_id"
    t.integer "user_id"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["plant_id"], name: "index_comments_on_plant_id"
    t.index ["user_id"], name: "index_comments_on_user_id"
  end
```

```
  create_table "gardens", force: :cascade do |t|
    t.string "name"
    t.datetime "start_date"
    t.datetime "end_date"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end
```

```
  create_table "plants", force: :cascade do |t|
    t.string "name"
    t.string "species"
    t.string "strain"
    t.string "type"
    t.string "sex"
    t.integer "time_until_harvest"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.integer "garden_id"
    t.index ["garden_id"], name: "index_plants_on_garden_id"
  end
```

### 4. Join table migrations for user_gardens, gardens_plants, and comments.

I learned that I needed to create two additional join tables for my has_many...through relationships: `user_gardens` and `gardens_plants`. 

```
create_table "user_gardens", force: :cascade do |t|
    t.integer "garden_id"
    t.integer "user_id"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["garden_id"], name: "index_user_gardens_on_garden_id"
    t.index ["user_id"], name: "index_user_gardens_on_user_id"
  end
```

```
  create_table "gardens_plants", force: :cascade do |t|
    t.integer "garden_id"
    t.integer "plant_id"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["garden_id"], name: "index_gardens_plants_on_garden_id"
    t.index ["plant_id"], name: "index_gardens_plants_on_plant_id"
  end
```

### 5. User model associations (has_many, belongs_to, and has_many...through)

Setting up the associations for the models took me two tries but I finally got what I wanted with the code below.

**User**

```
class User < ApplicationRecord

  has_many :user_gardens, class_name: "UserGardens"
  has_many :gardens, through: :user_gardens
  has_many :plants, through: :gardens
  has_many :comments, through: :plants
  
  validates_associated :gardens
  # has_many :plants, through: :comments
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable, :omniauthable

  def self.from_omniauth(auth)
    where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
      user.provider = auth.provider
      user.name = auth.info.name
      user.uid = auth.uid
      user.email = auth.info.email
      user.password = Devise.friendly_token[0,20]
    end
  end
end
```

**Garden**

```
class Garden < ApplicationRecord
  has_many :user_gardens, class_name: "UserGardens"
  has_many :users, through: :user_gardens
  has_many :plants
  has_many :comments, through: :plants
  validates :name, presence: true
  validates :start_date, presence: true

  def self.from_user(user_id)
    Garden.joins(:users).where(users: {id: user_id})
  end

end
```

**Plant**

```
class Plant < ApplicationRecord
  belongs_to :garden
  has_many :comments
  has_many :users, through: :comments

  validates :name, presence: true
end
```

**Comment**

```
class Comment < ApplicationRecord
  belongs_to :user
  belongs_to :plant

  validates :contents, presence: true

  def set_user!(user)
    self.user_id = user.id
    self.save
  end
end
```

I checked that my associations were all working by running various commands in rails console with `rails c` . These commands were based on user tasks and expected data in views.

### 5. RESTful CRUD routes

The routes for this app are manageable and not too complex. I know this is not a comprehensive set of routes and redirects, but for the purposes of building the minimal viable product, the routes are okay for the short term.

```
Rails.application.routes.draw do
  devise_for :users, :controllers => {:registrations => "registrations", :omniauth_callbacks => "callbacks" }
  devise_scope :user do
    get 'login', to: 'devise/sessions#new'
    get 'logout', to: 'devise/sessions#destroy'
  end

  devise_scope :user do
    get 'signup', to: 'devise/registrations#new'
  end

  resources :users, only: [:index, :show] do
    resources :gardens, only: [:index, :show, :new, :create, :destroy]
  end

  resources :gardens, only: [:index, :show] do
    resources :plants, only: [:index, :show, :new, :create]
  end

  resources :gardens, only: [:edit, :update]

  resources :plants, only: [:show] do
    resources :comments, only: [:index, :new, :create]
  end

  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
end
```

### 6. Controllers, views, and forms. Refactoring to use partials.

Building the controllers and forms was a lot of fun and I it taught me where to look out for pitfalls in future projects. I also started using partials for my forms and I really like using them to stay DRY.

```
<%= form_for @garden, url: user_gardens_path(current_user), method: "post" do |form|%>
  <% if @garden.errors.any? %>
      <div id="error_explanation">
        <h2><%= pluralize(@garden.errors.count, "error") %> prohibited this garden from being saved:</h2>

        <ul>
        <% @garden.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
        </ul>
      </div>
    <% end %>
  <%= render partial: "form", locals: {f: form}%> <small><%=link_to "Cancel", user_path(current_user) %></small>
```

### 7. Validation

I added validation on new and edit forms for gardens and plants toward the end of the project. 

```
if @garden.valid?
      @garden.users << current_user
      redirect_to user_garden_path(current_user, @garden) if @garden.save
    else
      flash[:error] = "<ul>" + @garden.errors.full_messages.map{|o| "<li>" + o + "</li>" }.join("") + "</ul>"
      render :new
    end
```

## Learnings:
* I need to really consider what tables and associations I want from the beginning. If I had planned a little bit better and researched how to accomplish the architecture I was aiming for, I wouldn't have needed to re-create my database a second time.
* I should have included validations earlier in the project in order to avoid simple logic errors within the controllers. I found myself getting errors when checking the views on test records I created with empty values that wouldn't have made it into the database if I had implemented validation early.
