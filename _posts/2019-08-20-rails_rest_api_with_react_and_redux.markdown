---
layout: post
title:      "Rails REST API with React & Redux"
date:       2019-08-20 18:29:41 +0000
permalink:  rails_rest_api_with_react_and_redux
---


## Putting it all together...ALL OF IT

In this project, I mainly focused on the React & Redux code rather than the Ruby / Rails. If you look at the [code on github](https://github.com/darrendaz/react-portfolio) will notice that there is a familiar [REST API ](https://github.com/darrendaz/rails-portfolio-project/tree/api-fix) and a [brand new REST API](https://github.com/darrendaz/rails-api) that I created just for this project. Follow the setup steps in the readme of the React project if you wish to try it out for yourself.

This was the most fun that I've had building a project. As simple as it looks, I actually failed a few times before even getting it to where it is now in the github repo because there sure were a lot of places for me to break things...

### Rails REST APIs:

At first I thought the original API would work and provide all the API endpoints needed to meet the project requirements, but I WAS VERY WRONG.

I found that I could not make a successful POST request to my API. Why? Because I didn't consider that I built that app with user logins and session authentication required to persist a new entry to the database. Rails took care of that behind the scenes and I was trying everything I could to get the post request to work for a simple comment feature. After googling and looking at the journey to the solution, I had to make a decision. Instead of replicating a user session with my React app, I adapted to what I had, which was working GET endpoints from my old API for Gardens and Plants and the ability to create a brand new API. It's realistic and worth practicing building an app that uses two or more APIs to pull data from, so why not.

In the end, I used fetch calls with Redux-Thunk  for my GET and POST requests. The GET requests were made to my Gardens and Plants endpoints from my original Rails app and the POST requests were made for the new Rails API to create and persist a Strain name to the database.

Rails-Portfolio-Project:

* Provides RESTful GET route for all garden objects and associated plants with their associated comments (I could have just made a single request to this endpoint to get everything for plants and comments as well, but I wanted to keep my concerns separate)
* Provides RESTful GET route for all plant objects and comments attached to them

Rails-API:

Used Faker gem to generate 25 seeds (dummy data) of strain names.

* Provides RESTful GET route for all strains
* Provides RESTful POST route for creating and persisting a new strain name entered by the user

### React and Redux

The React and Redux side of things equal parts rewarding and frustrating. Working with React and it's stricter friend Redux was very energizing and educational due to the popularity and abundance of helpful articles and threads. 

3 Container Components:

* Gardens container
* Plants container
* Strains container

All three of the above container components have a filter by name input field that utilizes state and props to filter and update the props of their respective child list components.

5 Stateless Components:

* Nav - Creates the navigation links using routes from React Router
* Home - Plain javascript object that returns a simple homepage with a greeting message
* Gardens List - Lists out 
* Plants List
* Strains List
* Garden Card
* Plant Card
* Plant Comments List

Redux:

Created the actions and reducers for gardens, plants, and strains.

Ex: strainsActions.js

```
export const fetchStrains = () => dispatch => {
  return fetch("http://localhost:4001/strains")
    .then(res => res.json())
    .then(strains => dispatch({
      type: "FETCH_STRAINS_SUCCESS",
      strains: strains.reverse()
    }))
}

export const setName = name => {
  return {
    type: "SET_NAME_SUCCESS",
    name
  }
}

const addStrain = strain => {
  return {
    type: "ADD_STRAIN_SUCCESS",
    strain
  }
}

const resetStrainForm = () => {
  return {
    type: "RESET_STRAIN_FORM"
  }
}

export const createStrain = strain => {
  return dispatch => {
    fetch("http://localhost:4001/strains", {
      method: "POST",
      headers: {
        'Content-type': 'application/json'
      },
      body: JSON.stringify({
        strain: {
          name: strain
        }
      })
    }).then(res => res.json())
      .then(strain => {
        dispatch(addStrain(strain))
        dispatch(resetStrainForm())
      })
  }
}
```

Ex: strainsReducer.js

```
const initialState = {
  name: "",
  strains: []
}

export default (state = initialState, action) => {
  switch (action.type) {
    case "FETCH_STRAINS_SUCCESS":
      return { ...state, strains: action.strains }
    case "ADD_STRAIN_SUCCESS":
      return { ...state, strains: [action.strain, ...state.strains] }
    case "SET_NAME_SUCCESS":
      return { ...state, name: action.name }
    case "RESET_STRAIN_FORM":
      return { ...state, name: "" }
    default:
      return state
  }
}
```

In the end, I'm glad that I was able to get everything working with what everything I've learned about Ruby on Rails, React and Redux libraries. However, I look forward to refactoring this code very soon and hopefully with some styling.



