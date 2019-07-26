---
layout: post
title:      "Rails Agro Project w/ JavaScript - Part 2"
date:       2019-07-25 22:30:50 -0400
permalink:  rails_agro_project_javascript
---

## About:

This is a continuation of the previous Rails Agro Project but with the addition of javascript files that use ajax, jQuery, and JSON to display data over Ruby on Rails erb. You can checkout my github repo for this project [here](https://github.com/darrendaz/rails-portfolio-project/tree/rails-agro-with-js).

## Steps:

At the beginning of this project I followed along with [this video](https://youtu.be/USe4L86MMHw) to get started with the initial setup.

### 1. Drop OG database and create new seed data

The first thing I needed to do was dropp my old databse and create clean seed data to work with as I write my new javascript. Things would have been really confusing and frustrating if the data was not controlled and consistent. 

Admittedly, I had to do the cycle of `rake db:drop` , `rake db:migrate`, `rake db:seed` and `rails c` running a few `create` and `destroy` methods a handful of times just to get my `seeds.rb` file working the way I wanted.


### 2. Modify existing controllers

Each of the controllers I wanted to use JS with required some code refactoring. Below are the code for the `index` and `create` actions as examples.

GardensController#index

```
def index
    if params[:user_id]
      @gardens = Garden.from_user(params[:user_id]).to_a
      respond_to do |f|
        f.html {render :index}
        f.json {render json: @gardens}
      end
    else
      @gardens = Garden.all
      respond_to do |f|
        f.html {render :index}
        f.json {render json: @gardens}
      end
    end
  end
```

GardensController#create

```
def create
    @garden = Garden.new(garden_params)
    if @garden.valid?
      @garden.users << current_user
      if @garden.save
        respond_to do |f|
          f.html {redirect_to @garden, notice: 'Garden was successfully created.'}
          f.json {render json: @garden}
        end
      end
    else
      flash[:error] = "<ul>" + @garden.errors.full_messages.map{|o| "<li>" + o + "</li>" }.join("") + "</ul>"
      render :new
    end
  end
```

### 3. Configuring new gems and dependencies for rails with JS

Once I refactored the controllers, I had to add a few things to my `.gemfile` :

```
...
gem 'active_model_serializers'
gem 'jquery-rails'
```

Then I needed to add `jquery` to my require list in `assets/javascripts/application.js`:

```
//= require jquery
...
```

Of course once I did those things I needed to do a `bundle install` in the command line in order for my changes to take effect.

### 4.  Generating and modifying serializers

Using `rails g serializer <serializer name>`, I generated the serializers for my garden, plant, and comment models. The next thing to do was align the associations in the serializers with the corresponding model associations. This step allows our associations and data to be serialized into JSON.

garden_serializer.rb

```
class GardenSerializer < ActiveModel::Serializer
  attributes :id, :name, :start_date, :end_date, :plants

  has_many :plants
end
```

plant_serializer.rb

```
class PlantSerializer < ActiveModel::Serializer
  attributes :id, :name, :species, :strain, :type, :sex, :time_until_harvest, :garden_id, :comments

  belongs_to :garden
  has_many :comments
```

comment_serializer.rb

```
class CommentSerializer < ActiveModel::Serializer
  attributes :id, :contents, :plant_id, :created_at

  belongs_to :plant
end
```

### 5. Writing the Javascript

Here's the fun part :) 

In my `garden.js` I needed to accomplish a few things:

* Write a function that makes an ajax request with jQuery to display the gardens on the index pages (All and User-filtered)
* Write a Garden class constructor that creates a new garden object from an ajax response
* Write a Garden.prototype function that takes the data attached to an individual garden object and returns the HTML to display those values.
* Write a callback function that takes the response data of the ajax request, uses the class constructor, and parses and pieces them together into injectable/appendable HTML.

```
$(function () {
  getGarden()
})

function getGarden() {
  const base_url = 'http://localhost:3000/gardens/'

  if ($("#gardens").data("id")) {
    const user_id = $("#gardens").data("id")
    const user_url = 'http://localhost:3000/users/' + user_id + '/gardens'

    $.ajax({
      url: user_url,
      method: 'get',
      dataType: 'json',
    }).done(data => gardensRequestCallback(data))

  } else {
    $.ajax({
      url: base_url,
      method: 'get',
      dataType: 'json',
    }).done(data => gardensRequestCallback(data))
  }
}

function gardensRequestCallback(data) {
  const element = document.getElementById("gardens")

  for (let item of data) {
    let garden = new Garden(item);
    let gardenListHTML = garden.gardenHTML()

    if (element) {
      element.innerHTML += gardenListHTML;
    }
  }
}

class Garden {
  constructor(garden) {
    this.id = garden.id
    this.name = garden.name
    this.start_date = garden.start_date
    this.end_date = garden.end_date
    this.plants = garden.plants
  }
}

Garden.prototype.gardenHTML = function () {
  let plantsHTML = this.plants.map(plant => {
    return (`
      <p><a href="/plants/${plant.id}">${plant.name}</a ></p >
      `)
  }).join('')

  return (`
    <div class="garden-data-container">
      <h3><a href="/gardens/${this.id}">${this.name}</a></h3>
      <p>Start date: ${this.start_date}</p>
      <p>End date: ${this.end_date}</p>
      <h4>Plants:</h4>
      <div>${plantsHTML}</div> 
    </div >
    `)
}
```


Next up `plant.js`

```
$(function () {
  getPlant()
})

function getPlant() {
  if ($("#plant").data("id")) {
    const plant_id = $("#plant").data("id")
    $.ajax({
      url: 'http://localhost:3000/plants/' + plant_id,
      method: 'get',
      dataType: 'json',
    }).done(data => plantRequestCallback(data))
  }
}

function plantRequestCallback(data) {
  const element = document.getElementById("plant")
  let plant = new Plant(data)
  if (element) {
    element.innerHTML += plant.plantHTML();
  }
}

class Plant {
  constructor(object) {
    this.id = object.id
    this.name = object.name
    this.species = object.species
    this.strain = object.strain
    this.type = object.type
    this.sex = object.sex
    this.time_until_harvest = object.time_until_harvest
    this.comments = object.comments
  }
}

Plant.prototype.plantHTML = function () {
  let plantCommentsHTML = this.comments.map(comment => {
    let timeStamp = new Date(Date.parse(comment.created_at))
    return (`
      <li>
        <small><b>${timeStamp}</b>: ${comment.contents}</small>
      </li>
    `)
  }).join('')
  return (`
    <div id="plant-data">
      <h2>Plant name: ${this.name}</h2>
      <p>Species: ${this.species}</p>
      <p>Strain: ${this.strain}</p>
      <p>Type: ${this.type}</p>
      <p>Sex: ${this.sex}</p>
      <p>Time Until Harvest: ${this.time_until_harvest}</p>
      <div id="plant-comments">
        <h3>Comments:</h3>
          <ul id="display-comments">
            ${plantCommentsHTML}
          </ul>
      <div>
    </div>
  `)
}
```


And finally, comment.js

```
$(function () {
  bindClickHandlers()
})

const bindClickHandlers = () => {
  if ($("#plant").data("id")) {
    const plant_id = $("#plant").data("id")

    $('#new_comment').submit(function (e) {
      e.preventDefault()
      if (this.comment_contents.value !== "") {
        const values = $(this).serialize()
        $.post(plant_id + '/comments', values).done(data => displayCommentsCallback(data))

        $('#new_comment')[0].reset()

        function displayCommentsCallback(data) {
          $('#display-comments').html("")
          const element = document.getElementById("display-comments")
          const newComment = new Comment(data)

          let plantCommentsHTML = newComment.plant.comments.map(comment => {
            const timeStamp = new Date(Date.parse(comment.created_at))

            return (`
              <li>
                <small><b>${timeStamp}</b>: ${comment.contents}</small>
              </li>
            `)
          }).join('')

          element.innerHTML += plantCommentsHTML
        }
      } else {
        alert("Please type something in before hitting Create Comment")
      }
    })
  }
}

class Comment {
  constructor(comment) {
    this.id = comment.id
    this.contents = comment.contents
    this.created_at = comment.create_at
    this.plant = comment.plant
    this.plant_id = comment.plant_id
  }
}
```


## Learnings:
* Having good data to work with from the beginning is important and time saving
* Javascript with RoR is actually pleasant to work with. Once I understood how Rails and Javascript play together, I was able to write and troubleshoot code much faster than using just Rails alone.
* It's a good idea to include logic that confines the behavior of your JS prevents it from firing on every page. This caused me to get console errors unrelated to the page I was working with.
* I have a lot more confidence making ajax requests



