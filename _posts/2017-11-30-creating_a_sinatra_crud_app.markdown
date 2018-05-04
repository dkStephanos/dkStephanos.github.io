---
layout: post
title:      "Creating a Sinatra CRUD App"
date:       2017-11-30 15:48:48 -0500
permalink:  creating_a_sinatra_crud_app
---

![](https://upload.wikimedia.org/wikipedia/en/2/2c/Sinatralogo.png)

**Fantasy Team Organizer**

For this project, the goal was to create a dynamic website with Sinatra and ActiveRecord. 

[Sinatra](http://sinatrarb.com/) is a domain-specific language written in Ruby that allows users to create a dynamic flow between different url paths, ultimately rendering a fully interactive website. [ActiveRecord](https://github.com/rails/rails/tree/master/activerecord) is a Ruby gem that effectively takes care of all our databse concerns. According to their readme: 

> Active Record connects classes to relational database tables to establish an almost zero-configuration persistence layer for applications. The library provides a base class that, when subclassed, sets up a mapping between the new class and an existing table in the database. In the context of an application, these classes are commonly referred to as models. Models can also be connected to other models; this is done by defining associations.

> Active Record relies heavily on naming in that it uses class and association names to establish mappings between respective database tables and foreign key columns. Although these mappings can be defined explicitly, it's recommended to follow naming conventions, especially when getting started with the library.

Using a combination of Sinatra and ActiveRecord to provide us our coding environment, the process of setting up a website is quite painless. First things first, we need to set up our Models and Tables.

**Models and Tables**

Using the Ruby gem [Rake](https://github.com/ruby/rake), the first step is to create migrations that will create our tables for our database. In this project, we will only require two tables, Users and Teams. A User has many Teams and a Team belongs to a user, which is an association that will be handled by ActiveRecord, provided we set up our tables right, and implement the correct macros. So our Users table just needs a username, and a password. We want our password to be protected, and will implement the Ruby gem [BCrypt](https://github.com/codahale/bcrypt-ruby), so the actual field for password will be named 'password_digest', so our Users migration will look like this:

```
class CreateUsers < ActiveRecord::Migration[5.1]
  def change
    create_table :users do |t|
      t.string :username
      t.string :password_digest
    end
  end
end
```

Our Teams table will hold all the actual data for our website, which will be the names of the players on the team. So we need a name and string field for each player position. Now, in order for ActiveRecord to implement our association correctly, our Teams table must also have a field for 'user_id', this way, when a Team is created, it is linked to the user that created it, allowing us to restrict the actions of users who did not create the team. As a result, our Teams migration will look like this:

```
class CreateTeams < ActiveRecord::Migration[5.1]
  def change
    create_table :teams do |t|
      t.string :name
      t.string :point_guard
      t.string :shooting_guard
      t.string :small_forward
      t.string :power_forward
      t.string :center
      t.string :sixth_man
      t.integer :user_id
    end
  end
end
```

The last step for setting up our Models, Tables and Associations is to create the actual model classes that correspond to our tables we just created. This is the easiest step, as we are essentially just turning over this responsibility to ActiveRecord, and we do that by making our models inherit from ActiveRecord::Base, and then using the appropriate macros. A macro is essentially a method that writes methods for us. So in order to create our Users have many Teams and Teams belong to a User association we are simply going to include the following macros: 'has_many :teams' within User and 'belongs_to :user' within Team. 

**Account Management**

The next goal for this project, is to provide a means of creating users, saving them to the database, and allowing users to log in and out of our site, therefore providing customized content and options. Since we set up our User model to have an encrypted password, all we have to do to log in/logout is set the 'user_id' field within the session hash. The session hash is simply a collection of data specific to individual 'visit' to our website. So when a user logs in, they enter a username and password, which is validated against the values in the database, and if valid, the user_id is saved within the session hash. When the user then logs out, our session hash is cleared. This means we have the ability to not only confirm a user is logged in before they access our site, but also gives us the functionality to tell exactly what user is attempting to create or edit our team objects. In order for all of this to work, we must have a .erb, or embedded Ruby file, and appropriate roots within our application controller. Those look like this:

```
<h3>Login</h3>
<form action="/login" method="POST">
  <input type="text" name="username" placeholder="Username:">
  <input type="password" name="password" placeholder="Password:">
  <input type="submit" id="submit" value="Log In">
</form>
```

```
get '/login' do
    if logged_in?
      redirect '/teams'
    else
      erb :'application/login'
    end
  end

  post '/login' do
    user = User.find_by(:username => params[:username])

  	 if user && user.authenticate(params[:password])
  			 session[:user_id] = user.id
  			 redirect '/teams'
  	 else
  			 redirect "/failure"
  	 end
  end
```

**CRUD Actions**

Now that we have our database set up, and our controller can handle users logging in and out of our website, it is time to give them something to do when we get there. The point of our site is to allow users to create, read, update and destroy Team objects that represent a user's fantasy basketball team. All of these actions will be handled by a team controller class which relies on Sinatra to provide a variety of get and post requests which navigate us through the site, collecting and setting data when appropriate, and ultimately rendering the .erb files which make up the visible aspects of our site. Here is the .erb, route and browser output for creating a new team:


**new.erb**
```
<h1>Create a new Team!</h1>

<form action="/teams" method="POST">
  <label>Team Name:</label>
  <input type="text" name="team[name]" value="Team Name">

  <br></br>

  <label>Point Guard:</label>
  <input type="text" name="team[point_guard]" value="Point Guard">

  <br></br>

  <label>Shooting Guard:</label>
  <input type="text" name="team[shooting_guard]" value="Shooting Guard">

  <br></br>

  <label>Small Forward:</label>
  <input type="text" name="team[small_forward]" value="Small Forward">

  <br></br>

  <label>Power Forward:</label>
  <input type="text" name="team[power_forward]" value="Power Forward">

  <br></br>

  <label>Center:</label>
  <input type="text" name="team[center]" value="Center">

  <br></br>

  <label>Sixth Man:</label>
  <input type="text" name="team[sixth_man]" value="Sixth Man">

  <br></br>

  <input type="submit" id="submit" value="Post Team">

</form>
```

**Routes in teams_controller.rb**
```
get '/teams/new' do
    if logged_in?
      erb :'teams/new'
    else
      redirect '/homepage'
    end
  end

  post '/teams' do
    valid_data = true
    params[:team].each do |key, value|
      if value == ""
        valid_data = false
      end
    end
    if valid_data
      @team = Team.create(params[:team])
      @team.user_id = current_user.id
      @team.save
    else
      redirect '/invalid'
    end

    redirect '/teams'
  end
```

**Browser Output**

![](https://lh3.googleusercontent.com/LvHoA2M40G34i4mbRgDwmBuoXJ4ea8EbL2mI_-rYDzqqcERo6zp7Kdu5iI6W6o-HZKYxuiglcH-0fq9nr1yDsG4lyr3zeX7IwiwIA3rIEWyPRGF26yEpej9oFuwsbUrpRTps3NfcEO6GW1SlKj8bIQLPrOW32BpJjdikqCFWKN71YJ3wKYV1E66uQi3gpax5sQgrIzofXg3KlgoN3kYw2FiYjWIYCWq-goHx8bExV911o_I9ULvh5cF9z4ATDfW8cPj9VEoDhV7R-3PzFXSbArcbP9Aoy0RjdIMHBww6kpyIx1njb5_snnOt2uCklVnXA6yjSVGjEY8DvHFg5Etm01oi3dKYwvzcrAelezHJGSKVatVXiwfAJqoG-ffpijzr16SiA0uTtlfhPm8qS5D1B7cOyu9eXBsnrFvBg2-R1UXWX18-cXuxnqsEZMb_31ZVNRNg5LN-0ZWWVB3hmumdDqi_ZkIvIuazI8nvCudBcuhv_1_sI1hByW6eCcnkNLaiJr-__O0d1EUKWn3n5sP8wsYW6gC0gptEXyXQ9wjQP7hTaiqc32ViUqVq4CuLX9VAtCT5Vg2B31jTpmq-vfUkLo9SJDwAUfQm61ap74zF=w301-h375-no)

A similar pattern is used for the other functions, where each action generally requires a .erb file and a get and post route for displaying and obtaining the data. The website itself has buttons which link to the different routes as appropriate, rendering errors or failure messages if a user tries to edit a team that is not their or enters invalid or missing data. Ultimately the Teams view and show page for an individual team look like this:

**Teams page**

![](https://lh3.googleusercontent.com/m-Gh0c9WUD3NeUALJf43TrjugxUHWhCXKDkVP6yUGOmpK2XtLyHqkMmd9p0M_hBL_k9INoZSuhdcXcJ4KMyLA6RbqJP1w6GJ-WLoMr2pQ2UKIvWu10VuaXwD6kIcvlssSjks1UWIh9xMFWcN7oATYKb90wY-yUaN1uVd2oJdWE1g5KarwTZ4BGKndlUO8pkhHNjDqZmZm3Bf0O2ax6OJE-hDYcvzKUdWAJoeWpRMO48qVH8Zn-IS_pXkP4wgNDg2mM3_T_rQPUNGMkalZ8GsrUa2xmYAt4N3_XsNL0FdmCAy-ZfmrD5fngr4MT91RXJXcX2k-0FMfZV2-smmFuQJP-Hvvzm2vdVmOmnvagc-AtsX3E8fRQaa-iYF8-UYSl32fXjz4-zes9So-ocBbh8Hk9gk0L8f4__a0-3XB2d9OUcuWJymR6q6tSDP4-xeUl90_WeAjnmp8P1D3PkrpU_T-t4iJIkSr7KkO6SbuHCQtDKpgUa0qcgTHFO7fM2Diqyvc_htsMhRfP30nK8dDd1218MZn7TM5XeOpVoDHj8pSXUVIzd5M1L3dH66wAODVSIpKC4Jl3XvCnugfokE0rTIKEchsiERvJoQN8fV6ll3=w270-h290-no)

**Team Show page**

![](https://lh3.googleusercontent.com/cRuJyUoD4N8juZ8s48op0hdjSdO1kgr_HG7EhymV4VckZSgVxn4WFV4M1ZB9rg5hXVyIgxSerfOTanckSp8LjyI-8Q6SeVEx74cty_xgcArTXok9rGIvnukZ09K93Z_sAgG5VdHWUPXJDF7jlsYJeZfNyH4xz6aJk3UhQlK4DzJ4GN62JHSWnVEQq3s0sIxHc-4fGvLxA-KxlTjQs2nilQB-HMcARLJFl5MWcjxOFGWDaJu7VTBOmRQ6eFw6NNXCNgJGMFYNw1l23RFYX10NpCHwv4RDj7sVuWduhj8DumO6kPu8P6Xi12_IkxEDcRyb_FgswIUCJOPUbZf9pk3hby32VsC5Rg620YGeZflLroR3bAXXIluBP46BuUrIv_vm6sdX0EWsRbZj3u6Pvn69J5e958pku7smL0Raht4FA0LV2FaNXERL8vbt3NnYajJlYH7yNja8hoewnRZvl6aP7wWui6qtmMFYZb4dvUOhdT3UTME7SnmFizgw59Co1nNNFIM8u3WaxXeXw57nF04r_1uIHb2bNOmGnR3Avfuwt8m6sRauLTpPKGgEBhKOyFvmqyD7QNTuqrYrxHqow6OsX8I5S-R6WvCT_SnVWjff=w247-h392-no)

**Conclusion**

At this point, we have created a collaborating website and database that relies on Sinatra and ActiveRecord to create object-oriented models linked to database tables which are dynamically associated. We have provided the html and logic to display our website, and to accept input and format it appropriately. We have created a means for users to sign up for our site, and access their created teams without fear of other users manipulating their data. We have also given users the ability to create, edit, update, and destroy the teams that they have made. So in conclusion, all it takes to create a dynamic website with Sinatra and ActiveRecord is to follow a basic pattern, beginning with creating your database, setting up your models and associations, writing your views, and then providing the controller logic to present all of this to your user. Don't just take my word for it, clone the repo off GitHub and try for yourself!


[Sinatra Fantasy Organizer!](https://github.com/dkStephanos/sinatraFantasyOrganizer)



