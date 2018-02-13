---
layout: post
title:      "React on Rails via Redux!!!"
date:       2018-02-01 14:35:23 -0500
permalink:  rails_and_javascript_a_json_love_story
---

The goal for today, is to set up the bare bones of a modern website, such that it can be expanded upon in the future. To make it future proof, we will start with a modern front end framework, React. React is a fantastic tool provided by Facebook, and it enables the use of intelligently rendering components to make sure our app runs as fast as possible, only updating what it needs to. React won't get us all the way there, we still need a backend framework to manage our database and supply our React front end framework with the data it needs. This is where our old friend Rails comes in. Rails essentially lives to be a backend framework now, and its plethora of gems and database options secures it as our choice. Finally, we need a way for React and Rails to communicate, they are in two different languages to be fair. Redux is the exact tool for such a job. By managing the state of our React app and sending asynchronous calls to our Rails server, Redux serves as the translator, negotiating on our behalf between the two servers.
# Setting up Rails
So first things first, lets create a Rails server! We can do that with the following command: 

```
rails new post-it-network-api - api --database=postgresql
```

We will use Postgresql as our database as it is compatible with Heroku. We also want to name our Rails app 'something-api' to indicate that this will be our backend server. We will also need to enable 'rack-cors' then, so we can make cross-domain requests between our servers. This means uncommenting the block of code found in 'cors.rb'. After that we can generate our first model, to keep things simple, we will start with a simple Post model, with a text content field. We will want to use the ActiveRecord Serializer gem to easily convert this model into a json object. Next we need to set up our routes, and we will want them to be namespaced under 'api' so our routes would look like this: 'http://localhost:3000/api/posts/1'. We can do that with the following code in 'config/routes.rb':

```
Rails.application.routes.draw do
  namespace :api do
  	resources :posts, except: [:new, :edit]
  end
end
```

Once we have our model and routes, we need to write the controller actions to supply the data to our front end. The important thing to remember here, is that since our Rails app is responsible for strictly backend logic, we have no views whatsoever. As a result, all of our rendering must be of json objects. We can test our API with apps like Postman which send AJAX requests to our server. Once we are done, our controller actions will look like these:

![Posts Controller](https://imgur.com/xetAMTc.png)
# Creating the React App 
Now that we have our backend ready to take and supply our data, we need to start implementing our user interface. In comes React. To create our client server, use the following command:

```
create-react-app post-it-client
```

Now, since React needs to be able to communicate with our Rails server, we will need to integrate Redux as we build up our client server. First things first, we need a component to display our posts. This has a lot of moving parts, so let's see what it looks like, and then explain what is going on. Here is our Posts container:

![Posts Container](https://imgur.com/SWvNceT.png)

So first things first, our Posts container actually renders a collection of PostCards, no pun intended. This is basically just a 'div' with a 'p' tag in it so far, but separating the view only component is important. Besides that, there are three big things going on. 

# Redux-ing it All Together

The first is the 'componentDidMount" function at the top. All React components have a lifecycle. They are mounted, rendered, destroyed, and React supplies functions that call in direct response to these lifecycle events. This allows us to do things like make asynchronous calls for data to our backend, without putting the rest of our application in a perpetual chokehold. 

Second is the 'mapStateToProps' function towards the bottom. This is where Redux comes in. All React components also have a 'state'. This state is tied to individual properties, like our post content in this case, and changes in state is how React knows to re-render our component. Managing state, especially with a  backend server, can become complex and overwhelming, and that is why we use Redux. Redux essentially wraps our whole React app in a Provider with a store of the current state of the application. When we call 'mapStateToProps', we are asking our Redux provider for the current properties for our component.

How does it know how to ask the store for the current state though? This is where the third important feature comes in, the 'connect' function. Connect does just what it sounds like, it connects our component to the store. By subscribing to the state of the entire application, we can just ask for our state when we need it, and the render appropriately. We have established the connection between the store and the component, but we still need a way to tell the store what we want. 

In comes our reducer. When we want to communicate with the store, we 'dispatch' and action, and that action is interpreted and the corresponding events are triggered. We specify what these actions are and what to do when they occur by writing a reducer. A reducer is essentially just a switch statement for actions. Here is what one looks like:

![Post Reducer](https://imgur.com/owvWhoo.png)

It has an initial state, and a case for each of our actions. Our reducer always returns the state in some form, depending on what we are trying to accomplish. So when we dispatch and action, it is caught by our reducer which determines which path to take, and then returns us some meaningful state with which we can render data. In order to integrate our reducers with our store and our React app, we need to do a little set up, which looks like this:

![Store creating](https://imgur.com/r40ZIjX.png)

# Adding Posts to a Page
Now we have a backend server supplying us with JSON objects from our database via Rails. We also have a user interface setup with React connected to our state and our database with Redux. So lets throw this all together and make some Posts! We will need a PostForm that is connected to our store and capable of sending POST requests to our Rails API. Here is what that component looks like:

![PostForm Container](https://imgur.com/ymSxU7a.png)

Now that we have our form container that is connected to our store, we can place it in our React app. When the form is submitted, we will intercept the event, serialize the form data, and make a POST request to our Rails API, which will then inject our data into our database, storing it as a new instance of the Post class. Our Posts container then fetches the raw JSON of our Post, converts it into properties to be passed down to our PostCard, and we see our new post rendered in the browser. We have successfully created a server to handle our data, a server to handle our user interface and integrated them together. This leaves room for expansion and multiple additional features, but for now, just look at all those brand new posts!

![Adding Posts to Page](https://imgur.com/NUkLI8B.png)

