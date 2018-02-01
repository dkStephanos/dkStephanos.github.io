---
layout: post
title:      "Rails & Javascript: A JSON Love Story"
date:       2018-02-01 19:35:23 +0000
permalink:  rails_and_javascript_a_json_love_story
---

![album show page](https://i.imgur.com/awSV1zm.png)

For this project, I set out to take my functional Ruby on Rails Album Organizer application, and implement a little Javascript-esque magic with the help of JSON and JQuery. The ultimate goal is to allow certain data to be updated automatically without the need for a page refresh. This includes things like displaying a form to add new artists or albums directly on the index page, rendering the newly created instance upon creation, as well as navigating through our album of songs without having to redirect. In order to make this possible, however, we are going to need a way to access data in a meaningful and constructive way that allows us to offset responsibility for certain tasks to our Javascript files. This is accomplished by setting up our own API of JSON responses.

# Setting up the API
API stands for Application Programing Interface, and is essentially the way that a programmer views and interacts with our website as opposed to a typical user, who just uses the browser views. API's can be set up in various ways, but perhaps the most popular is with JSON. JSON stands for Javascript Object Notation, and is an easy choice to pair with Javascript and AJAX. AJAX stands for Asynchronous Javascript and XML, and is essentially the means of interacting with the API, which itself can be implemented in myriad ways, we will be using jQuery. So, what does all this vocabulary mean? Basically, our website will have an API, which is really just a standard of rendering objects in a raw JSON format, such that it can be accessed with AJAX calls and then used in our Javascript. Simple, right?

Actually, setting up the API is one of the similar tasks, as they can be implemented with our old friend ActiveRecord. ActiveRecord implements a Serializing interface. A serializer is exactly what it sounds like, it take our raw Ruby object, and converts it into a string that can be sent across the internet, in our case, a JSON string. Once you have generated your serializers, and specified the attributes to include when serializing, we just have to render the JSON strings in a way they can be fetched with AJAX. Rails is smart enough to be able to handle this for us, if we prompt it to that is. By simply checking the format of the request in our controller actions, we can choose to render the native view via its .erb file, or we can render JSON. So once we have our serializers, and our controller set up, when we visit a link like 'album/1/song/4.json", we get something that looks like this:

![](https://i.imgur.com/p9ZFmdN.png)

# Javascript Model Object
What do we do with that, you might be pondering. That is where our Javascript files come in. We can use jQuery to submit AJAX requests to the JSON links that we just created. When we do this, we get back the JSON object, like the one above. From here, we can manipulate the data as we see fit, selecting elements from our DOM to update appropriately. One of the easiest ways to implement this is by converting our JSON string back into something that resembles the Ruby object it came from, and that is where our Javascript Model Object comes in: 

![](https://i.imgur.com/D2lFdq1.png)

This is a Javascript Model Object for our Song. It is declared like a function, because in Javascript, everything is an object, including functions. Inside the function call, we can set attributes to call upon later. We can even extend the functionality by adding methods to the prototype of the object. So in this case, we are adding a method that converts time from seconds, into a string of minutes and seconds, formatted to be displayed in the DOM. Javascript Model Objects aren't required to perform some of these actions, but the parallel with our Ruby objects and the ease of organization make them a great choice.
# Dynamic Forms
So now that we have a reliable API which we can pull data from, lets try and post data to our API. This isn't actually terribly difficult, and simply requires a different AJAX call with data formatted in line with our API. What would be great, however, is if we could pull data we need from our API, render a form when requested, create a new instance of our object and inject that into our DOM, all without a page refresh. Seem ambitious? Let's give it a shot. First things first, we need a form. I went ahead and added one to my index page, and set it to be hidden initially, and then revealed when the user clicks the 'add Album' button. Here is what that looks like:

![](https://i.imgur.com/PyPaWhu.png)

Once our form is there, we just need to attach a listener to it, so that when the form is submitted we can intercept the event, and perform our Javascript tasks. To do this, we need to bind an event to the 'submit' action of the form. Now, since the DOM is complicated, we want to make sure we don't attach more than one handler, so we unbind and then bind our event handler, to make sure there is only ever one applied. Once we have an event handler, we need to prevent the default action of the form, submitting and redirecting, and instead pull the values from the form fields and serialize them into valid JSON. Once we have this string representing the data to be injected into a new Album, we submit a post request with jQuery with our serialized values, and upon completion, add our new album into our index, and hide the form. Here are the methods that handle this:

![](https://imgur.com/gLXfqZg.png)

# Show Page Navigation
![](https://i.imgur.com/nV5AfQS.png)

Perhaps the most difficult feature for me to implement was navigating the song show page. I wanted you to be able to scroll through an album, looking at each song's data, and allowing you to favorite/unfavorite songs, or even edit/delete them if you created them. In order to do this, we need navigation links bound to event handlers. Since we don't want to be able to navigate past the end of an album, or before the album, we need these navigation links to be dynamically updated. Furthermore, since songs can be deleted, we can't rely on a continuous chain of song id's. We will need to fetch the album data via an AJAX request, and use the song id's found in that JSON string for our navigation. Once we have the collection of ids, stored as an array, we know which song to attach to the 'next' and 'previous' links. I did this by having a data-id field on our links that was updated as the song changes, that way we always know where we are, and where we are going. The actual updating of all the fields in the DOM gets a little complicated, but this is what the set up looks like:

![](https://i.imgur.com/2qgXnJI.png)

# Conclusion
Ultimately, I would consider this project a success. By implementing our own API, not only have we extended the usability of our site, we have also allowed for completely new features that were before unattainable. The html and css are all our own still, but we are moving away from that being necessary. If we have reliable JSON data and flexible templates, we can move almost all our functionality away from redirects and view rendering, and into our Javascript. This gives us a more professional presentation, with easier adaptability and can even allow other developers to interact with and extend our site.





