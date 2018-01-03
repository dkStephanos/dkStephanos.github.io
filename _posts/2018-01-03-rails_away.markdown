---
layout: post
title:      "Rails Away!!!"
date:       2018-01-03 17:06:06 +0000
permalink:  rails_away
---

![](https://images.pexels.com/photos/6966/abstract-music-rock-bw.jpg?w=1260&h=750&auto=compress&cs=tinysrgb)

For my first app written in Ruby on Rails, I set out to make a basic album organizer, that would allow a user to create an account to manage their music. This allowed me to flex some of the new knowledge I have accumulated in a practical setting that involves user authentication, data validation, CRUD actions in Rails, separating logic and concerns between models, views and controllers, as well as some basic CSS. Before I started worrying about what my album show page or new song form looked like, I set out to build an app simply capable of managing my different users.

# Devise and OmniAuth

I chose the ruby gem Devise to handle my user authentication. Devise is essentially its own application that resides within your website. It has its own views, routes and generators, and as a result, really provides the heavy lifting for your account management. Devise does this by adding a set of Modules to your user model, each of which extend varying functionality. Once added, the User model has the ability to be authenticated by the database, initialize new users from session data, remember users already logged in and even track them when they are. Devise also adds its own routes and views, so once installed, all you have to do is provide links to the content it generates.

This is great for handling users created on our website, but a little added functionality would be great! This is where OmniAuth comes in. Another Ruby gem, OmniAuth is actually built into Devise, but serves a different purpose. OmniAuth allows external authorizations, or in other words, the ability to log in with a website that is not our own. OmniAuth requires additional gems depending on the website, Facebook, Google, Github, etc.. This requires a bit of set up, however, first of which is setting up an initializer in the file *config/initializers/omniauth.rb*. This looks like:

```
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :facebook, ENV['FACEBOOK_KEY'], ENV['FACEBOOK_SECRET']
end
```

This informs our Rails app that we are using middleware created by OmniAuth that validates with Facebook. The KEY and SECRET in the code refer to data from Facebook directly, so we need to head there. Facebook has its own developer site with lots of functionality for developers that either use Facebook for authentication or more advanced purposes. Once on the site, you can navigate to the *My Apps* section and *Add a New App*. Once you have created your app, you can go to *Facebook Login* and enter in your local url under *Site URL* and then your local url plus */auth/facebook/callback* which is the default callback endpoint. After all of that, we now have the KEY and SECRET for our app. Now all that is left is placing them in a .env file and using the gem dotenv-rails to handle it. Now we can set up our session controller and place the appropriate links, and when we navigate away from our site, everything is taken care of for us.

# CRUD with Nested Resources
Now that we can handle Users created on our site or logging in from elsewhere, its time to give them something to do. The point of this app is a little simplistic, in that we are only managing Artist/Album/Song/Genre data. In order to flex our ability to nest resources within our url, I chose to associated everything down from Artist using *have_many* and *belongs_to* relationships. This means that Artist is associated to its Songs through and Album, and an Album may have many Genres through its Songs. In order to keep these relationships in mind, it makes sense that when we visit an artist's album on our site, the album is actually listed under our artist. For example, selecting the first album under the first artist would result in a path like this:

```
amazonaws.com/artists/1/albums/1
```

This would allow us to have access to the album's artist directly from the params hash, but we don't have access to those kinds of routes by default, we must tell Rails that we are using nested resources. Thankfully, everything in Ruby looks a lot like what it sounds like, so all we have to do is add the following code to our routes.rb file:

```
resources :artists do
    resources :albums, :only => [:index, :show, :new, :edit]
  end
```

This adds a whole new set of routes such as *new_artist_album* and *edit_artist_album* that keeps the idea of an album belonging to an artist present throughout our application. This also allows us to automatically set data in the views for albums based on the artist id found in the url. Using the author_id like this means that we must include it in our views and controllers so that we don't lose it along the way and it can be of use to us. This means that we must include it in our strong params declaration and either use a field on our view, which can be hidden, so that we don't lose that data when we navigate to a new url. We also need to add some checks in our controller to make sure that the artist or album listed in the url is actually the instance associated to the data we want. Otherwise, a savvy user could change the id in the url, and make changes to data they shouldn't have access to. Once we have handled these exceptions, however, all that is left is to place the correct links in our views, and pass on creating and editing responsibilities to our controllers, which should be pretty familiar at this point. For example, the check to make sure we are editing what we are supposed to be:

```
def edit
        if params[:artist_id]
            artist = Artist.find_by(id: params[:artist_id])
            if artist.nil?
              redirect_to artists_path, alert: "Artist not found."
            else
              @album = artist.albums.find_by(id: params[:id])
              redirect_to artist_albums_path(artist), alert: "Album not found." if @album.nil?
            end
        else
           set_album
        end
    end
```

# Scope and Custom Attribute Writers
At this point, we have a user authentication system granted to us by Devise and OmniAuth. We also, thanks in part to nested resources, have an efficient CRUD system that can handle our Artist, Album and Song models quite well, keeping the basis of their associations present throughout. Perhaps the most difficult and interesting aim of this project, however, was not just to allow users to login and perform basic CRUD actions, but to add some flexibility and functionality. This is accomplished by two things in particular, which are both related to our Song model. What if we wanted the ability to not only create and save Songs and Albums by Artist, but also denote that certain songs are of importance. Saving favorite posts or items or even songs is nearly universal in today's online experience, so lets pass that on to our users as well. In order to do this, our Song model will require a boolean field, *is_favorite*. This will allow us when we log in, to scan all songs that belong to us, filtering out the ones we have chosen as favorites.

```
<div class="songList">
    <h2>Favorite Songs</h2>
    <% if current_user.favorite_songs.empty? %>
        <p>No Favorited Songs</p>
    <% else %>
        <ul>
            <% current_user.favorite_songs.each do |song| %>
                <li><%= link_to ("#{song.name} [#{song.time}]"), song_path(song) %></li>
            <% end %>
        </ul>
    <% end %>
</div>
```

The other feature we wanted to add was the ability to create a new Genre when submitting a song. For the rest of our models, creating a new instance is traditional and straight forward. We click a link that takes us to a form which submits the data to a controller which then creates and saves our new instance. Genre, however, is a little different. Our Artist and Album models have many genres associated with them through Song, but unlike songs and albums, a Genre can belong to multiple Artists. This means that we don't need to be as protective when creating new Genres, because while it is associated with our other models upon creation, we might want to use the same Genre tag for another song or album or even artist. This makes perfect sense, but requires something a little different, which is a custom attribute writer. This means that within the song model, when we pass in the data for the Genre, instead of simply searching for the Genre, we perform a *find_or_create_by* action, which will return an instance of Genre whether or not it existed before our search. This means our view must look a little different: 

```
<div class="field"> <%= f.label :name, "Name:" %><br>
    <%= f.text_field :name %></div>
    
    <div class="field"><%= f.label :length, "Length(in seconds):" %><br>
    <%= f.text_field :length %></div>
    
    <div class="field"><%= f.label :album_id, "Album:" %><br>
    <%= f.collection_select(:album_id, Album.all, :id, :name, {:selected => params[:album_id]}) %></div>
    
    <div class="field"><%= f.label :genre_name, "Genre:" %><br>
    <%= f.text_field :genre_name %></div>
    
    <br></br>
    
    <div class="field"><%= f.label :is_favorite, "Add to Favorites:" %>
    <%= f.check_box :is_favorite %></div>
```

Notice how all of the fields besides genre have the actual names of the data we are setting. The name for example corresponds to *:name* and the length to *:length*. The Genre, however, is denoted as *:genre_name* and that is so we can catch our custom attribute writer. We also have to include the *:genre_name* in our strong params so it is passed down via mass assignment. In our Song model, we have the following method which handles our Genre creation:

```
def genre_name=(genre)
       self.genre = Genre.find_or_create_by(name: genre) 
    end
```

# Conclusion
At this point, our website is fully up and running. We have left it open to expansion, and you can probably already think of new features to add. What is most important though, is that we have a valid user authentication system that can handle requests to external websites. Once logged in, we can manage a collection of associated models, and create update and destroy instances of those models, provided we have the correct authorizations. We also have the ability to filter data based on user selected fields, and even proactively create new instances of our models when the system dictates its necessary. These are the fundamental of a Ruby on Rails application, and from here, you can really go wherever you want!

