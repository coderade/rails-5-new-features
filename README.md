# Rails 5 new features

This project show the new features, improvements and other important changes to Ruby on Rails version 5. It will shows how to use the most important features of Rails 5, including Action Cable, Action Controller Renderer, Turbolinks, the Active records, Attributes API and how to use Rails exclusively as a JSON API.

This project will discuss too the many other smaller changes in improvements such as the Rails command Router, new Date and Time methods, Secure Tokens and changes to parameters.

In addition, it will show the features that are being deprecated or completely removed. This is an important topic if you have an existing project that uses any of these features.



## Table of contents

- [About Ruby on Rails 5](#about-ruby-on-rails-5)
- [Major features](#major-features)
    - [Action Cable and Websockets](#action-cable-and-websockets)
        - [Uses for Action Cable](#uses-for-action-cable)
        - [Differences between the HTTP Model and the Action Cable](#differences-between-the-http-model-and-the-action-cable)
            - [HTTP Model](#http-model)
            - [WebSockets Model](#websockets-model)
        - [Why WebSockets?](#why-websockets)
        - [Why Not WebSockets?](#why-not-websockets)
        - [Using ActionCable](#using-actioncable)
            - [ActionCable Terminology](#actioncable-terminology)
            - [Example](#example)
    - [ActionController::Renderer](#actioncontrollerrenderer)
        - [Overview](#overview)
        - [Examples](#examples)
    - [Rails as JSON API](#rails-as-json-api)
        - [Why Rails as JSON API?](#why-rails-as-json-api)
        - [How do we use it?](#how-do-we-use-it)
    - [Turbolinks 5](#turbolinks-5)
        - [Why Turbolinks?](#why-turbolinks)
        - [Using Turbolinks](#using-turbolinks)
            - [Turbolinks Partial Replacement](#turbolinks-partial-replacement)
    - [ActiveRecord Attributes API](#activerecord-attributes-api)
    - [ActiveRecord::Relation#or](#activerecordrelationor)
        - [Guidelines](#guidelines)
- [Improvements](#improvements)
    - [Rails Command router](#rails-command-router)
    - [Application Record and Application Job](#application-record-and-application-job) 
    - [Integer methods: #positive? and #negative?](#integer-methods-positive-and-negative)
    - [Date and time improvements](#date-and-time-improvements)
    - [Relations in batches](#relations-in-batches)
    - [ActiveRecord Secure Tokens](#activerecord-secure-tokens)
    - [Rails Generate command](#rails-generate-command)
    - [Migration class](#migration-class)
    - [Abort ActiveRecord Callbacks](#abort-activerecord-callbacks)
    - [Changes to parameters/params](#changes-to-parametersparams)
    - [Enumerable#without](#enumerablewithout)
    - [Enumerable#pluck](#enumerablepluck)
    - [ArrayInquirer](#arrayinquirer)
    - [Per-Form CSRF tokens](#per-form-csrf-tokens)
    - [belongs_to Requires Parent](#belongs_to-requires-parent)




## About Ruby on Rails 5

* First major release in three years - That was for the version four and it's been about a year and a half since Rails 4.2 came out.
* Faster
* Less memory usage
* Less time doing garbage collection (GC)
* Requires 2.2.2 or greater
* [GC of symbols (Incremental GC)](https://www.sitepoint.com/symbol-gc-ruby-2-2/)
* **Incremental GC: ~2x faster** - Previously, the Ruby would periodically go through all the objects in memory, mark the ones that were still fresh or being used and then make a second pass to remove the unmarked objects from memory.
The problem with this is that that GC sweep through memory takes some time, it can slow down your code while it's happening. Ruby 2.1 improved the speed of this process somewhat by classifying objects in memory based on the likelihood they would need to be garbage-collected, but a large collection could still slow things down. Well, now, in Ruby 2.2, we get incremental GC and the basic idea is to break up the scanning and removal of objects so that the process happens incrementally, not all in one big sweep, so while the total work and the total time may still be the same, it's broken up among many smaller operations.
* **Optimizes common operations** - The core team has optimized common operations, the code has fewer dependencies on other code in libraries and there are fewer object allocations to memory. Now, that may seem like a small point that doesn't make much difference, but, as one example, link and URL generation in Ruby on Rails 5 is 44 percent faster.
* Fewer dependencies
* Fewer object allocations
* **Development environment is faster** - The development environment uses the Puma web server now, instead of using [Webrick](https://github.com/ruby/webrick) and the development mode used to check the modification time of all of your files in your project to know if anything had changed too, so that it could automatically reload the development environment for you. But now, there is a file system monitor, which notifies Rails whenever something changes. There is no more requirement to check the modification times of all the files. And that makes development feel snappier when you are working. And on top of these performance improvements.


## Major features

### Action Cable and Websockets

* Framework for working with Websockets
* Allows real-time features using a constant connection between server and clients
* Comprised of server-side Ruby and client-side JavaScript
* Uses Redis pubsub to track communications

#### Uses for Action Cable

* Streaming content such as video
* Interactive content such as online games, or any similar application where you're swapping mini messages per second between the client and the server
* Realtime communications such as chat

#### Differences between the HTTP Model and the Action Cable 

##### HTTP Model

* Data exchanged is independent of connection (each time a communication needs to happen, a new connection is created)
* Wrapped data: request headers + data
* Headers can 1-2KB large (even if the data is tiny) - includes things like cookies, that the browser has stored. Even if the data is tiny, it still has to wrap it in these headers and the server overhead goes both ways.
* Independence comes at cost of overhead
* Request-response cycle communication

##### WebSockets Model

* Data exchanged is dependent on connection
* Headers sent only at the start -  we don't have to have that same amount of overhead every time we swap a message back and forth.
* Only data sent afterwards
* Dependence allows significantly less overhead
* Full-duplex communication - Instead of having this request response cycle that takes place with the HTTP model, now with WebSockets, we can exchange information back and forth and we can swap information back and forth at the same time, because the connection is open, the client can send information to the server at the same time as the server is sending information back to the client,


#### Why WebSockets?

* Less data overhead
* Faster
* Realtime
* Full Duplex

#### Why Not WebSockets?

* Monopolizes a serve connection - No one else can talk to the server across that connection while it's in use
* Even when nothing much is happening and a lot of data's not being sent
* HTTP request-response cycle can serve more clients - So when I make a request for a webpage, the server sends me the webpage and then while I'm reading the webpage and deciding what I want my next decision to be, my next request to the server, it can go ahead and serve other clients in the meantime.
* No page caching - Different of the HTTP requests, with something like a webpage, that can be cached and sent very quickly back to the client. With WebSockets we have to dynamically respond all the time and in fact, most applications just simply
* Most applications do not require realtime communication


With WebSockets we have to dynamically respond all the time and in fact, most applications just simply do not require real-time communication, so ActionCable and WebSockets is a great solution for people who need real-time communication, but if you don't need it then you're probably better going off with a more traditional HTTP model.

#### Using ActionCable

##### ActionCable Terminology

* Cable - single connection between a server and a client that's one single client, so when we open a connection between them, we call that a cable.
* Channel - stream of data to which clients ca subscribe
* Subscriptions: a client's currently subscribed channels
* Broadcast: message sent on channel to subscribed clients

##### Example

If we wanna use ActionCable, the first thing that we need to do is make sure that it's enabled in our routes, so in your routes file, you wanna make sure that the line that says mount `ActionCable.server` is uncommented and available for you to use. You must include this if you wanna have ActionCable.

By default it's going to point to the URL/cable, but you can configure that to something different if you'd like. This is the route that we're going to use for communication with ActionCable.

```ruby
# config/routes.rb
Rails.application.routes.draw do
    # ...
    
    mount ActionCable.server => "/cable"

end    
```

Next, we're gonna need to create our channel and a new Rails 5 app should have a directory for channels where you can put this code and you're gonna create channels in much the same way that you normally create controllers, so while they're not exactly the same, you'll find that they are similar, so here you can see I've got a class defined for `ChatRoomChannel` that inherits from `ApplicationCable Channel` and then you can see that I have a subscribed action and a speak action.

```ruby
# app/channels/chat_room_channel.rb
class ChatRoomChannel < ApplicationCable::Channel
    
    def subscribed
        stream_from('chat_room_channel')
    end

    def speak(data)
        ActionCable.server.broadcast('chat_room_channel',
          :message => data['message'])
    end

end    
``` 

There will usually be an unsubscribed action as well, subscribed and unsubscribed are the two standard actions. These two actions do whatever code you want to do whenever a user subscribes or unsubscribes from a channel. Often what you wanna do when they're subscribing is to start streaming data to them across the channel and you can see that that's what I'm doing in my subscribe method when I'm calling stream from. For this overview I created The speak method, that is a custom action of my own creation, on it I've added just a basic example, when the speak action is called, then the ActionCable server is going to broadcast a message on the chat room channel and you can see that I'm passing in whatever I want that data to be.

Anyone that's streaming from that channel anyone who's subscribed, will be sent that message immediately, once that we have our chat room channel set up and we have subscribed, unsubscribed and any of the custom actions that we wanna have inside there. 

Then we're ready to set up the JavaScript side of things. In your JavaScript, you'll need to set up the cable like so, on the bellow example you see I have variable for App and then I'm setting `App.cable` equal to `ActionCable.createConsumer`, this gives our JavaScript object all the features that it needs to communicate with the server.

```javascript
// app/assets/javascripts/cable.js
//= require action_cable
//= require_self
//= require_tree ./channels
 
(function() {
  this.App || (this.App = {});
 
  App.cable = ActionCable.createConsumer();
}).call(this);
``` 

Once we have it set up, we can use it to set up a room on our app object, which creates a new subscription to the chat room channel. Here you can see I'm setting `App.room` equal to `App.cable` subscriptions create, so I'm creating a new subscription and I'm assigning it to room. I'm gonna call it `ChatRoomChannel` and inside you'll see that I'm defining a number of functions. I've got functions for connected, disconnected, received and speak.

```javascript

//app/assets/javascripts/cable/subscriptions/chat.coffee
//Subscription JS
App.room = App.cable.subscriptions.create(
    "ChatRoomChannel", {
        connected: function(data) { },
        disconnected: function(data) { },
        received: function(data) {
            alert(data['message']);
        }
        speak: function(message) {
            this.perform('speak', {message: message});
        }
    }
);
``` 
**connected**, **disconnected** and **received** are standard functions in `ActionCable` that you'll probably always want to have. Whenever I'm connected, the JavaScript code in the connected function will execute, whenever I'm disconnected the code in the disconnected function will execute and the received will gets called anytime data is received from this channel. 

In the above example, in my received function I'm just calling a simple JavaScript alert to display whatever message is received. You could instead replace data on the page, or add the data to some existing content, for example, if this was a real chat room, you'd probably want to append the message received to the end of the things that already been said in the chat room.

The Speak is a custom function of our own creation, it can be called anything we like, but notice that what it does, is it calls perform speak. This is how we call the controller action speak that we created earlier, send it data and that's the message that we wanna speak. Remember, that action just broadcast our message to anyone who subscribed to this same channel.

Now we've defined all of our Ruby actions on the server side and all of our JavaScript functions on the client side. 

In order to use it, we just have JavaScript call our speak function, like so, so we call `App.room.speak`, anywhere in our code and after we provided the message it goes through the code that we want. 

```javascript

//app/assets/javascripts/communication.coffee
//Communication JS


App.room.speak('Hello ActionCable');
//Calls this.perform('speak');
//Calls ChatRoomChannel#speak
//Calls ActionCable.server.broadcast

//Message goes to the streamed WebSocket
//Triggers App.room.received on all subscribers
``` 

Let's review what the above code looks like: it calls `this.perform('speak')` and it passes along our message That then in turn calls `ChatRoomChannel` speak. Which in turn calls `ActionCable` server broadcast and sends our message out to that channel and anyone who subscribed to it is going to get it.

It's gonna be streamed to that WebSocket, once that's received by the client, it's gonna trigger `App.room.received` on all the subscribers, which in turn calls alert and alerts with our message, so that we see what the message was.

Now this is a very simple example, meant to just give you the flow of how things work, there's a lot more that you can do and a lot of configurations to help you to do them.

If you wanna experiment or learn more, a great resource is the Rails ActionCable examples that are on [github](https://github.com/rails/actioncable-examples) and official Action Cable [documentation](http://guides.rubyonrails.org/action_cable_overview.html).



### ActionController::Renderer

* Render templates independently of controllers.
* Useful for ActionCable

#### Overview

Another major feature of Ruby on Rails 5, is ActionController Renderer. ActionController Renderer allows us to render templates, but to do it independently of our controller, it's particularly useful for things like ActionCable.

We can render bits of content, that we broadcast to all our subscribers and we don't have to load up all of the action controller code, with parts that we don't need like cookies and redirects, we can just use the rendering feature on its own.

#### Examples

The way that you use it, is very straight forward and it's just like you would render inside a controller, so instead of just calling render, we now call `ApplicationController.render` and then tell render what we wanna render.

On the bellow sample a template called `products/index` will be rendered and it's gonna look for that template in its normal place, it's gonna look for a template called index in the views directory inside the products directory.

```ruby
ApplicationController.render(
    :template => 'products/index'
)
```    

You can also use the shorthand form for this, that you're probably more familiar with and that is just to call `ApplicationController.render` and then the template name that we want.

```ruby
ApplicationController.render('products/index')
```

Another nice shorthand is that if instead of calling render on ApplicationController, we call it on one of our custom controllers which inherits from ApplicationController. Then we don't even have to provide the directory for where to find the template.

It knows that it's going to look in the products directory, because we're working with the ProductsController, so that's where it's going to look by default, for this index template and just like our controllers we don't have to just render the html.

```ruby
ProductsController.render('index')
```

We can render a JSON, for example. If we wanted to provide a JSON back as a snippet to action cable, we can do that just like this:

```ruby
ApplicationController.render(
    :json => Product.all
)
```

The other possibilities for things you can render are templates, actions, partials, file, inline, plain, text, html, json, js and xml.

Just like you're used to having in your controllers, you can pass local variables to the templates, just like you normally would.

On the below example, where I'm rendering a partial for `products_product` and I'm providing local variable values for product and for user. 

```ruby
ApplicationController.render(
    :partials => `products/_product`,
    :locals => {
        :product => Product.first,
        :user => current_user
    }
)
``` 

There's only one thing that's slightly tricky about this and that is that in your controller, it's common practice to set up instance variables, which are then automatically going to be bound to the ERB template that we render, so that they're available for use inside that template.

That binding doesn't happen automatically with renderer, but render lets us pass instance variables for binding to the template, using assigns, so on the below example, you can see that I'm passing in both locals and assigns. This would mean that inside that partial I would be expecting product to be a local variable and the user would be an instance variable.

```ruby
ApplicationController.render(
    :partials => `products/_product`,
    :locals => { :product => Product.first },
    :assigns => { :user => @user }
)
``` 

In case the difference between assigns and locals is not completely clear, here's another example that shows them both being applied to the rendering of a bit of inline ERB, so my template is that inline ERB.

You can see that in the first blank I have an instance variable for version and a local variable then in the second blank for adjective, so I'm using assigns to set the instance variable, I'm using locals to set the local variable. 

```ruby
(
    
    :inline => "I think Ruby on Rails version <% @version %> is <% adjective %>!",

    :assigns => { :adjective => 'sweet' }
    :locals => { :version => 5 },
)
``` 

Overall, rendering works pretty much the same way that it does inside our controllers, what's different in Rails 5, is the fact that we've gained the ability to do this rendering outside of controllers. We don't need a controller in order to render code, we can do it inside our channels.

We could do it inside our jobs, our mailers, inside our rake tasks. There's all sorts of places that rendering can now take place without having to have all that controller code loaded in.

For more information about Renderers see the Ruby `ActionController::Renderer`[documentation](http://api.rubyonrails.org/classes/ActionController/Renderer.html).

### Rails as JSON API

* Based on Rails::API [gem](https://github.com/rails-api/rails-api)
* API for clients to access data and application logic
* Backend for client-side frameworks (Client eg.: JS, Flash, Iphone, or Rails)

This feature almost made it into Rails 4 back in 2012, but instead it became a stand alone Ruby gem, which has now been incorporated into Rails 5.

You should note that it is possible to have an existing Rails application which doubles as a JSON API. In other words it serves up both HTML and JSON. But that's not what this overview is, this will talk about making a Rails app which is exclusively an API.

We're gonna be setting up our application from the start so that it omits a lot of the normal Rails application code.

#### Why Rails as JSON API?

* **No need for templates, HTML, CSS and JavaScript** - there's no needed for templates, HTML, CSS and JavaScript, all that can be left out and when we use our generators in Rails it's gonna skip generating the views, the helpers and the assets whenever we create a new resource.
* **Most application logic is outside the View layer** - most of the application logic is performed inside our controllers and models and not inside the view, so we can let the client just simply be our view layer for us
* Less controller and middleware code - when we go to create a new resource, the Rails as an API is going to leave out modules which are primarily useful for browser applications, like cookie support. It won't put jQuery or turbolinks into our gem file because it knows we won't be using them.
* Configured for API-style applications
* Familiar Ruby framework, RubyGems, Bundler
* Sensible defaults and generators
* Keep key features: security, routing, caching, error handling, logging and etc.
* Development and test environments - the test is an important aspect too, so we can continue to write test for our application in a way that we're used to do our Rails apps.


#### How do we use it?

To create a Rails API application all you need to do when you create a new Rails app is to add the `--api` option to the end, as the following command:

```
rails new my_backend_api --api
```

The biggest difference is this single line, as we can see on the below example, inside the `config/application.rb` file there's a new line that says `config.api_only = true` and that's what tells Rails to be in API mode in everything that it does. 

```ruby
# config/application.rb
module MyBackendApi
    class Application < Rails::Application

        config.api_only = true    
    end    
end    
```

Every time you use a generator, every time you do anything in your application, it's going to be in API mode, if you're converting over from an existing application, then this is a line that you'll need to add.

Another difference is in our controllers, we still have controllers, but we don't want all the HTTP parts that we aren't going to be using, so the `ApplicationController` is going to inherit from the class `ActionController::API`. 

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::API

end    
  
``` 

Normally it would inherit from `ActionController::Base`,this change leaves out code like cookies and sessions, which we don't need, but still gives us other parts of the controllers that we do want, such as before actions, security methods, page caching. 

The controllers that we generate are going to inherit from `ApplicationController`, which in turn inherits from the API.

```ruby
# app/controllers/notes_controller.rb
class NotesController < ApplicationController
    
    def index
        @notes = Note.all
        render(:json => @notes)
    end

    def show
        @note = Note.find(params[:id]) 
        render(:json => @note)
    end

    # also: create, update, destroy
    # without: new, edit
end    
  
``` 

So our controllers are going to work the same way and the generated content of our controllers will also be different.
Notice in the above example that these controllers rendered JSON resources by default, they also include create, update and destroy actions, but unlike regular controllers they don't automatically have a new and edit action, because typically those are just HTML pages for submitting forms and an API client would be expected to take care of that business for us. 

Our routes still use resourceful routes and we define them the same way, but if you take this and then run `rails routes` on it you'll see that the new and edit routes that are normally present when doing restful routing are no longer included with the API version.

```ruby
# config/routes.rb
Rails.application.routes.draw do

    resources :todos
    
end    
```

We don't need them, in the same way that we don't need the actions that correspond to them, you can add them back if you decide you do, but they're not there by default. 

With Ruby on Rails 5, Rails developers who want to create backend applications where Rails serves as a JSON API will find it much easier than they have in the past and it will also omit a lot of the application code that you don't need to make your application as lean and fast as possible.

For more information of how use Rails for API-only Applications, see the official [documentation](http://edgeguides.rubyonrails.org/api_app.html).


### Turbolinks 5

Turbolinks was first introduced in Ruby on Rails 4. This version of Turbolinks has now been renamed as [Turbolinks Classic](https://github.com/turbolinks/turbolinks-classic) and version five, which comes with Ruby on Rails 5, is a ground up rewrite with a new flow and new events, but which still uses the same core idea as Turbolinks Classic.

The new version is faster than the old one and also offers the ability to do partial replacements of webpages. We'll take a look at that feature in a moment.

#### Why Turbolinks?

* Performance of a single-page application - that's part of what JavaScript frameworks like to boast about
* No client-side JavaScript framework required - you can work directly with HTML pages in your existing Rails application. 
* Links do not load completely new pages
* Fetch new content with Ajax; swap `<body>`; merge `<head>` - it fetches the page using asynchronous JavaScript, also known as Ajax and then it takes the returned page and it replaces the body of the current page with the new body and it merges the content of the page header with the existing page header. 
* CSS and JS is retained (does not need to be reloaded or re-rendered)
* Content is still full HTML page, not fragment or JSON
* Current browser scroll position is maintained
* Still changes browser's URL and history - the back button will work as expected
* Much faster, can feel almost instantaneous (<1 second)


#### Using Turbolinks

To use the Turbolinks, we need:

* first to add the turbolinks to Gemfile: `gem 'turbolinks'`
* Run the `bundle install` command
* Add to JS manifest: `//= require turbolinks`

You don't have to use Turbolinks all the time, you can have some links that opt out of it. 

```html
<a href="/some_url" data-turbolinks="false"> 
    This link will not use Turbolinks
</a>
```

As the above example, there I have a link that I've marked with `data-turbolinks=false` and this now becomes a link that will not use Turbolinks when it loads. Otherwise, all your other links will automatically use Turbolinks and you'll get that speed boost as a result.

There is one gotcha when working with Turbolinks and that is: that it's very common for people to write JavaScript that either activates on loading the window or when the document is considered ready and that's not going to be true with Turbolinks anymore, because our document and our window are already loaded.

```javascript
//Javascript
window.onload = function(){
    // ...
}

//jQuery
$(document).ready(function(){
    // ...
});
```

All we're doing is swapping data into the existing document, so if you find that your JavaScript isn't firing anymore, then instead you wanna use the EventListener, turbolinks, load events and if you'd listen for turbolinks load, your JavaScript will fire again.

```javascript
//Javascript
document.addEventListener('turbolinks:load',function(){
    // ...
})

//jQuery
$(document).on('turbolinks:load', function(){
    // ...
});
```

```javascript
//Javascript
window.onload = function(){
    // ...
}
```

The turbolinks load event fires once on the initial page load and then again after every turbolinks visit. 

##### Turbolinks Partial Replacement

The first version of Turbolinks simply replaced everything within the body when a link was clicked, that kept the HTML head, CSS and JavaScript, but the bulk of the page was still reloading and you can continue to use Turbolinks in that way and you'll get a big speed boost by not reloading that extra stuff, but the new version of Turbolinks that's in Ruby on Rails 5 has an additional feature.

You can reload only parts of a page by marking content as either **permanent** or as **temporary**. 

Look the below example, let's imagine that we have a page and inside the body I have three divs, one with an ID of nav, one with an ID of flash and one with an ID of comments. 

```html
<body>
    <div id="nav" data-turbolinks-permanent>
        <!-- will not change after the initial load -->
    </div>

    <div id="flash" data-turbolinks-temporary>
        <!-- update with every request unless told to 'keep' -->
    </div>
    
    <div id="comments">
        <!-- changes on each page load or when targeted -->
    </div>
</body>
```

You can see that I've marked the nav as being data turbolinks permanent. That means that this div is not going to change after the initial load by default. Then I've got div id flash marked data turbolinks temporary and that's going to be updated with every request unless I specifically ask it to stick arounds and then I've got a normal div which I've just called comments and that's typically going to change on each page load.

Using the content marked up appropriately let's switch over to the controller and see how some of the different rendering options affect those tags. 

```ruby
# Keep #nav, update all other parts of page
render('comments/index')

# Update #flash & #comments, keep #nav & other parts
render('comments/index', :change => 'comments')

# Keep #nav & #flash, update all other parts of page
render('comments/index', :keep => 'flash')

# Update everything, including #nav
render('comments/index', :flush => true)
``` 

For the first example, if I just do a simple render, let's say I render `comments/index`, it's going to keep the nav div around, it's not going to make any changes to it, it will leave it in place because I've marked it permanent. It's gonna update all the other parts of the page.

In the second example, I've specifically targeted comments by using change, you can also use append and prepend in the same way, so I'm targeting comments, therefore it's going to update the flash and the comments.
It updates the comments because I targeted it and it updates the flash because I marked that it's being temporary, so it's still going to be refreshed each and every time, but the nav and all other parts of the page, any other content on the page is left alone. 

Like the third example, if I instead tell it to keep the flash, well then the flash will stay around, so now in the third example you'll see that it's gonna keep the nav, because I've marked it permanent, it's going to keep the flash, because I've explicitly asked it to, but all other parts of the page are gonna be updated.

On the Last example, you can see that I'm updating everything including the nav by using flush true. This is still using Turbolinks, it's not opting out of Turbolinks, it's just telling it that all the parts of the page should be replaced, the entire body gets replaced in that case.

The below example show how the things are done inside your controller. You can also do the same thing in your Javascript, this is essentially what's happening under the hood, when the Ajax makes this call. That is that we're calling `Turbolinks.visit` comments and then we can provide any of those options we just saw.

```ruby
# Keep #nav, update all other parts of page
Turbolinks.visit('/comments')

# Update #flash & #comments, keep #nav & other parts
Turbolinks.visit('/comments', :change => 'comments')

# Keep #nav & #flash, update all other parts of page
Turbolinks.visit('/comments', :keep => 'flash')

# Update everything, including #nav
Turbolinks.visit('/comments', :flush => true)
``` 

Change, keep, flush, as well we can use append and prepend and those do the exact same thing. If you wanna find out more about Turbolinks. Learn more about some of the configuration options that are available to you. 

You can find out more about it at the Turbolinks official [repository](https://github.com/turbolinks/turbolinks).


### ActiveRecord Attributes API

Another major feature of Ruby on Rails 5, is the ActiveRecord Attributes API, that can not be confused in any way with the Rails as a JSON API we wrote above on this doc. This is different, this is about working with ActiveRecord Attributes in our models. 

In our models, ActiveRecord automatically detects attribute types from the underlying database - for example, if our database has a varchar type for a column, then it sees that as being a string. If it sees an int type, it sees that it's being an integer a decimal becomes a float, a tinyint becomes a boolean, and a date time becomes date time.

What the attributes API does is it gives us a standard way to override the type detection and other defaults that are built into Rails, and the way we do it is with a new method called attribute, so we have attribute and we pass in three different arguments:

```ruby
attribute(name, cast_type, options)
```
The first is the name of the attribute. The second is the cast type, and the third is any options that we want to provide.

For example. Let's say that we create a new table in our database called fabrics, and in that table I'm adding a new column which is gonna be a decimal column called quantity.

```ruby
create_table :fabrics do |t|
    t.decimal :quantity
end    
``` 

That would normally give me a floatback when I access it as an attribute in my fabric class, but when I tell it to use integer instead.

```ruby
class Fabric < ApplicationRecord
    attribute(:quantity, :integer)
end    
``` 

Now will always give me an integer and never a float, so you can see that example below, when I then call `fabric = Fabric.new`.

```ruby
fabric = Fabric.new(:quantity => 3.9)

fabric.quantity.integer? 
# => true

fabric.quantity 
# => 3 
``` 

I provided a float but instead `fabric.quantity` is now going to be cast as an integer, and when I ask it for its value, the value is three. Not 3.9, because it converted it from a float into an integer.

It can also override the default values that are set in the database, so for example, if I have my fabrics database and I have a sting column called name, and it has a default database value of database default, so the database is going to want to set that value. 

```ruby
create_table :fabrics do |t|
    t.string :name, :default => 'Database default'
end    
``` 

By default Rails is gonna pick up that same default value, but if I instead tell the attribute that it'll have a different default, then I can override whatever is there.

```ruby
class Fabric < ApplicationRecord
    attribute(:name, :string, :default => 'New default')
end    
``` 

So now when I call `fabric = Fabric.new`, and I call `fabric.name`, I get back New default, not database default anymore, and also these attributes do not need to be even backed by a database column.

```ruby
fabric = Fabric.new(:quantity => 3.9)

fabric.name
# => 'New default'
``` 

You can create your own attributes and define them in the same way, and affect how Rails will cast those values. You can also define your own types. Just make sure that the type you create implements all the methods that ActiveRecord type. 

The easiest way to do that, is to inherit from an existing type or from ActiveRecord type value. Then you can override the parts that you want and call super, to let the parent class do the rest. The key methods to override are cast, serialize, and de-serialize.

```ruby
# inherit existing class or ActiveRecord::Type::String

class BackwardsType < ActiveRecord::Type::String
    def cast(value)
        new_value = value.reverse
        super(new_value)
    end 
end

ActiveRecord::Type.register(:backwards, BackwardsType)
``` 
On the above example, I'm overriding cast so that it reverses any string that it's given before it puts it in the database. Once we create our type, we just need to register it. Calling ActiveRecord `type.register`. I'm gonna give it a name of backwards, so that's the name of my type. Instead of string, integer, and decimal.

Now it's gonna have a type of backwards which will be my new class, so I can for example, create my table of users. I give it a string, I call it backwards name, and then I use my attributes API to set backwards name using the backwards type.

```ruby
create_table :users do |t|
    t.string :backwards_name
end  

class User < ApplicationRecord
    attribute(:backwards_name, :backwards)
end
```

So, user = User.new backwards name "Coderade". and what is the value of backwards name? It's gone through my class, it's been typed in the way that I've told it. Which in this case, I told it to reverse the letters. 

```ruby
user = User.new(:backwards_name => 'Coderade')

user.backwards_name
# => 'edaredoC'
```
It's a little bit of a silly example, but I think it makes the point of how you can really do anything you want inside these class types. A more common use for writing your own type would be to write the serialization and de-serialization yourself, so for example, you might wanna be able to provide JSON to an attribute and then have that JSON be serialized in a certain way before being put into the database and the be de-serialized back to JSON when you pull it back out of the database.

Rails gives a sensible default for our attributes. The ActiveRecord Attributes API gives us a way to override those defaults.
 
You can find out more about the  ActiveRecord Attributes API at the your Rails documentation [page](http://api.rubyonrails.org/classes/ActiveRecord/Attributes/ClassMethods.html).

### ActiveRecord::Relation#or

Ruby on Rails 5 adds a new method, which is ActiveRecord::Relation#or. This has been a long-requested feature, and it finally arrives in Rails 5.

First, let's take a look at a query that uses two where clauses to reduce the results that are being returned using and. 

```ruby
Post.where("status = 'published'").
    where("published_on = '2017-08-29'")
```
On the above example, I have post where status equals published and where published_on is 2017-08-29. Now I put the and in there, because that's what's really happening when we write the SQL.

```sql
SELECT * FROM posts WHERE (status = 'published') AND (published_on = '2017-08-29')

-- Select all from posts where status equals published and published on equals the date that I've given it. 
```

Now daisy-chaining these kind of scopes together is common in Ruby on Rails, and it works well. But what about the case in which we don't want to scope them further by using and, but we want to broaden our scope by using **or**. In other words, where one thing is true or something else is true. This has always been a problem in Ruby on Rails. And up until the Rails 4, you either had to write the SQL yourself or used a third-party library to help you. 

But starting on Ruby on Rails 5, we have new method with is or. 

```ruby
Post.where("status = 'published'").or(
    Post.where("status = 'pending'")
)
```
The way it works is like this; you can see I have post where status equals published, and then what I chain onto the end of that is OR.

 The resulting SQL looks like this:
 
```sql
SELECT * FROM posts WHERE (status = 'published') OR (status = 'pending')
 
-- Select all from posts where status equals published or status equal pending. 
```

The **or** method expects an active relation as an argument. Any active relation will do, so you can use scopes as well, and you can keep chaining them together as long as you always provide an active relation as the argument. Now this isn't going to work for complex queries, but it is going to be a big help in simple cases.

```ruby
Post.published.or(Post.pending)
Post.published.or(Post.pending).or(Post.recent)
```

#### Guidelines

There are some guidelines that you need to keep in mind, though.

* First, the argument that you pass  Must be an `ActiveRecord::Relation` (query or scope) - That's why I had to use `Post.where` on the above example, again, because we needed to created a `ActiveRecord::Relation`
* The arguments must be the same model - So if my exterior query has to do with post, then the thing that goes inside or should also be regarding Post. 
* And the arguments must differ only by their `#where` or their `#having` clause - that's because that's what's going on under the hood. `ActiveRecord::Relation#or` is taking any where or having clause that we pass in on a scope, and it's appending them to the existing where and having clause using or.
* Should not use `#limit`, `#offset`, or `#distinct` - you can still use limit, offset, and distinct on the exterior scope, but not on the relation that's being passed into or.

If you keep these guidelines in mind, I think that you'll find that or provides a handy tool to our Active Relation query toolbox.


## Improvements

### Rails Command router

One of the most noticeable improvements in Ruby on Rails 5 are the changes that have been made to the Rails Command Router. When working with Ruby on Rails 5, from the command line we typically issue one of two kinds of commands, either `rake` or `rails`. So, for example, we do `rake db:migrate`, `rake test`, `rake spec`, or even custom Rake tasks, and then we have `rails new`, `rails generate`, `rails server`, `rails console` and many others. The problem is that these are inconsistent in the way that they are used and the code is even maintained in different directories.

This becomes especially confusing for beginners who are trying to remember when do I use Rake and when do I use Rails and they don't really care about the technical differences between the two, they just want a consistent way to interact with the application. 

So starting in Version 5, we're going to use Rails for everything, now it's going to feel odd to veterans of Ruby on Rails because we've typed `rake db:migrate` so much. It's going to take some getting used to to type `rails db:migrate`, but that's what's it's going to be. You're going to use Rails to run all of the old Rake tests.

What Rails is going to do under the hood is that it checks for those key Rails scripts, so `new`, `generate`, `console`, `server` and etc. and then, if what you're asking for isn't one of those key scripts, then it's going to send anything else on to Rake and let Rake look for it, so that means that any of the existing Rake tasks, any custom Rake tasks that you have, and third-party library Rake tasks, those are all going to use Rails, because Rails will just send them on to Rake. So that means that Rake tasks do still exist, you can still write them, and then can even still be used as dependencies.

You're just not going to access these scripts from the command line anymore, instead you want to use Rails for that. This change may feel odd during the transition, but over the long term this is a change for the better.


### Application Record and Application Job

Ruby on Rails 5 adds a couple of new classes to the framework to help us out, `ApplicationRecord` and `ApplicationJob`. They are customizable super classes for all of your models and your jobs. These classes are very similar to the way `ApplicationController` and `ApplicationHelper` work, we put all the code that's common to our records and our models in ApplicationRecord and anything that's common to our jobs should go in ApplicationJob, that allows us to keep from overriding the base classes directly.

The files for ApplicationRecord and ApplicationJob are going to be created by default in a new Rails 5 project, but they're not required to exist, they're really just there as part of the framework to help you out in the same way that you could do away with ApplicationController and everything would still work just fine.


In a new Rails 5 project it will create a file in app slash models slash application underscore record that looks something like the bellow example. It will have a class ApplicationRecord which inherits from ActiveRecord Base. 

```ruby
# app/models/application_record.rb
class ApplicationRecord < ActiveRecord::Base
    self.abstract_class = true
end    
``` 

Then you'll on the next example, when we work with our models those models are now going to inherit from ApplicationRecord not ActiveRecord Base.

```ruby
# app/models/product.rb
class Product < ApplicationRecord
    self.abstract_class = true
end
``` 

On the Rails 5, they don't have to. They could still inherit from ActiveRecord Base and everything would still work fine, but now we have this intermediary super class where we can put code that's common to all of our models.

Notice that ApplicationRecord has a very important line in it that says `self.abstract_class = true`, that tells Rails don't assume that there's an underlying table called AppllicationRecords which can be queried. Instead this is just a utility class that has no database table behind it. Even if you never have common code that you want to put in to ApplicationRecord it's good to go ahead and follow this new model that the Rails framework wants us to use.

The same thing is true for ApplicationJob. ApplicationJob is a class that inherits from ActiveJob Base and then each of our jobs would inherit from ApplicationJob instead of ActiveJob Base. 

```ruby
# app/jobs/application_job.rb
class ApplicationJob < ActiveJob::Base
    
end
``` 

```ruby
# app/jobs/remove_old_carts_job.rb
class RemoveOldCartsJob < ApplicationJob
    
end
``` 

There's no need to use `self.abstract_class = true` because out jobs don't have databases behind them anyway.

To demonstrate why this small change is a significant improvement to Rails. Let's compare version four and five. Here in Rails 4 I would have a module called `ActsAsCoolFeature` and inside it might be whatever methods I have that do that cool feature.

On the bellow example, I would have a module called ActsAsCoolFeature in Rails 4 and inside it might be whatever methods I have that do that cool feature.

```ruby
# Rails 4
module ActsAsCoolFeature
    def do_a_cool_feature
    end
end

ActiveRecord::Base.include(ActsAsCoolFeature)
``` 
As we can see on the above example, if I want to add that into my models and I want to have it in all of my models I would have ActiveRecord Base include ActsAsCoolFeature. Now, all of my models have ActsAsCoolFeature built into them.

Pretty cool, right? Except that this technique is called [monkey patching](http://stackoverflow.com/questions/394144/what-does-monkey-patching-exactly-mean-in-ruby) and it refers to modifications of a class at run-time with the intent of changing their behavior. 
By patching ActiveRecord Base directly we're patching it for everyone that uses it. If we're running a small app that's not a big deal, but if we're using gems, plug-ins, engines, or writing complex code, each one of those is potentially expecting to have a clean ActiveRecord Base to work with and our module could interfere with their work.

Instead, in Rails 5 with ApplicationRecord we have the ability to include the class in our intermediary class, ApplicationRecord, and not in ActiveRecord Base. ActiveRecord Base stays a pristine class that everyone can use. Our features included, but only for our models which inherit from ApplicationRecord. 

```ruby
# Rails 5
module ActsAsCoolFeature
    def do_a_cool_feature
    end
end

class ApplicationRecord < ActiveRecord::Base
    self.abstract_class = true
    include ActsAsCoolFeature
end
```

The same thing is true for ApplicationJob and the desire to keep a pristine version of ActiveJob Base around. So while it may seem like the Rails framework has just thrown a couple of extra files into a default application, they're there for a good reason and I encourage you to use them.


### Integer methods: #positive? and #negative?

Another improvement in Ruby on Rails five is the addition of two new methods on the integer class. `positive` and `negative` and they work exactly as you'd expect. 

When we call positive question mark or negative question on an integer, they return either true or false, `1` would be considered positive, `-1` would be considered negative and `0` is considered neither positive nor negative, as you can see those below results:

```ruby
1.positive?
# true

0.positive?
# false

-1.positive?
# false

1.negative?
# false

0.negative?
# false

-1.negative?
# true
```

While these are two very useful methods, they are also going to be a superfluous part of Rails soon, and that's because these methods are built-in methods in Ruby 2.3 which was released in December of 2015. 

So really what Rails is doing here is just offering these methods early for Ruby 2.2 users and if you remember that's the base requirement for using Ruby on Rails five is Ruby 2.2.2, so if you meet that requirement you still have access to these methods and if you use Ruby 2.3 then those are going to be built-in methods for you.


### Date and time improvements

Ruby on Rails 5 provides new methods for working with date and time.

Ruby on Rails has new methods for next day and previous day (`#next_day`, `#prev_day`), these do the same thing as yesterday and tomorrow, which were existing methods, but, because their support for next week and next year it makes sense to have a parallel version for day. It was always a little odd that you could ask for next year and next week, but you couldn't ask for next day, you had to ask for tomorrow instead. So now they exist and you can use either one.

Ruby on Rails also gained the concept of a weekday and a weekend, So now you can ask for next weekday and previous weekday (`#next_weekday`, `#prev_weekday`) and you'll get the following day unless it's on a weekend. So, for example, if it's a Friday and we ask for next week day, we don't get back Saturday we get back Monday. 

To correspond with those, we have some query methods, on weekend and on weekday (`#on_weekend?`, `#on_weekday?`) which will tell us whether something is on a weekday or weekend. 

As I mentioned, there are some existing methods for next week and previous week (`#next_week`, `#prev_week`), those have just gained a new ability, which is they have the `same_time` argument. 

Then, time gets days in year as a new method (`Time.days_in_year(year)`), it already had a `days_in_month` method, this matches that and it defaults to the current year.

Let's say that I have the current time. And, let's say that the current time is Friday, March 18th, 2015 at 12:26 UTC. So if I ask for `Time.current.next_day`, then it returns tomorrow. It returns the following day, which is Saturday the 19th.

```ruby
Time.current
# => Fri, 18 Mar 2016 12:26:11 UTC +00:00

Time.current.next_day
# => Fri, 19 Mar 2016 12:26:11 UTC +00:00

Time.current.prev_day
# => Fri, 17 Mar 2016 12:26:11 UTC +00:00

Time.current.next_weekday
# => Fri, 21 Mar 2016 12:26:11 UTC +00:00

Time.current.prev_weekday
# => Fri, 17 Mar 2016 12:26:11 UTC +00:00
```

The previous day returns Thursday, March 17th, both at the same time of day. If I ask for next weekday, then because it was on Friday, it doesn't return Saturday to me it jumps to Monday and, it returns Monday at the same time to me.
If I ask for previous weekday, well, the previous weekday is the same thing as previous day so it returns Thursday, if the current time had been a Monday, then previous weekday would, of course, return a Friday.

We have the ability to query those with on weekday and on weekend the current time is on a weekday, it's true. 

```ruby
Time.current
# => Fri, 18 Mar 2016 12:26:11 UTC +00:00

Time.current.on_weekday?
# => true

Time.current.next_day.on_weekday?
# => false

Time.current.on_weekend?
# => false

Time.current.next_day.on_weekend?
# => true
```

The Current next day, which is Saturday, is that a weekday? No, it's not. But if we ask on weekend, `Time.current` is not on the weekend but `Time.current.next_day` which is Saturday is on the weekend.

We also can take a look at the changes to next week. So again, let's say I had the same time. 

```ruby
Time.current
# => Fri, 18 Mar 2016 12:26:11 UTC +00:00

Time.current.next_week?
# => Fri, 25 Mar 2016 00:00:00 UTC +00:00

Time.current.next_week(:thursday)
# => Fri, 24 Mar 2016 00:00:00 UTC +00:00

Time.current.next_week(:friday, :same_time => true)
# => Fri, 25 Mar 2016 12:26:11 UTC +00:00

```
Look the above example, If I ask for next week, I get the next week, I get Friday, but notice what happened to the time, the time got reset to midnight. That's what next week does, next week also accepts an argument. So, we can ask for a certain day of the week.

`Time.current.next_week(:thursday)` returns Thursday of next week. But, again, the time goes to Midnight. That's the way it behaved in Rails four and the way it still works in Rails 5. 

However, in Rails 5, we now get a new option that we can pass in of same time true, in order to use this `same_time` argument, you must provide the day of the week that you wanted. It's not optional, you can't leave it out. So in this case, I have time dot current dot next week and I'm asking for Friday with the same time option.

I mentioned already that time had a days in month method built in, it allows us to has for how many days are in a month. So, `Time.days_in_month(1)` tells me how many days are in the month of January, which is 31 and I can ask it for February and I can provide an optional year and it'll tell me how many days are in that year's month. In February, that makes a difference because sometimes it has has 28 and sometimes it has 29 days, I can also do the same thing with days in year. 

By default, it'll return the current year, if I pass in an argument of 2016, I get back the same result. But, if I were to pass in 2015, which is not a leap year, then I get back 365.


```ruby
Time.days_in_month(1)
# => 31

Time.days_in_month(2, 2017)
# => 28

Time.days_in_year
# => 365

Time.days_in_year(2016)
# => 366

```

Each one of these examples is a small improvement to Rails, but taken all together they make date and time easier to work with.


### Relations in batches

Ruby on Rails 5 gives us the ability to work with Active record relations in batches. To understand how it works, let's first look at find in batches.

```ruby
Product.find_in_batches do |batch|
    sleep 5
    batch.each do |product|
        product.recalculate_inventory
    end
end

```

Which is an older method that was in Rails 4, and continues to exist in Rails 5, it looks like the above example. Product and then the `find_in_batches` method and then a block of code that I wanna perform on each batch. 

The default batch size is 1,000 records, or you can pass in batch size as an option if you want something different. The way that this works, is that Ruby on Rails makes a query to the database and finds the first 1,000 records, it brings them back, instantiates them into objects and puts them in array and renders them up to my block of code.

My particular block of code here in this example is going to sleep for five seconds and then iterate through the products in that batch and tell each product to recalculate its inventory. This technique works great when we have expensive or long running processes that would monopolize the resources of a server. 

By sleeping for five seconds between batches, it gives the server a change to breathe, to catch up, to answer other people's requests, and do their jobs, instead of letting my code monopolize its resources.

In many cases, this technique works great. However, there are some cases where it doesn't work. Look the below example:

```ruby
Product.find_in_batches do |batch|
    batch.update_all(:on_sale => false)
end
```
Let's imagine that I call product.find in batches and then what I wanna do to each batch after I sleep for five seconds is to tell it to update all the records in that batch with on sale false.
The `update_all` is a great method to use when we wanna update a lot of records, because what it does is it sends off a single SQL query to the database, telling it to update all of these records with this value.

The reason why this doesn't work, is because update all is something you would call in a class or on an active record relation or a scope. Instead, in this case, batch is an array. It's an array of objects from the database, so the query has already been made, the objects have been created and they're put into this array and I can't call update all on it. Now, I could go through each of those products in that array and tell each and every one to update its attributes, so that on sale was equal to false, but if I did that then each batch would send off 1,000 SQL queries to the database.

Where as update all would update all 1,000 records with one SQL query which is much more efficient, I'd really like to be able to use it, but it doesn't work with find in batches.

What Ruby on Rails 5 offers us, is a new method which is called `in_batches`.

```ruby
Product.in_batches do |batch|
    sleep 5
    batch.update_all(:on_sale => false)
end
```
It's the same thing, but it doesn't have find at the beginning, It makes it easy to remember. 

The difference here, is that product.in batches does not fire off SQL and go out and retrieve the objects and instantiate them and put them in an array, Instead it returns a relation to batch and that way when we're inside the block I can call `batch.update_all` and it fires off that single SQL query to update all 1,000 records at one time.

Now I also have the ability to tell the batch that it should go out and retrieve those records and do something else with them, if I want to, but it doesn't presuppose that I want to do that, it waits and allows me to make that choice inside the block and that makes in batches a nice option for us to have.


### ActiveRecord Secure Tokens

Ruby on Rails 5 provides a new feature to make it easy to create unique random tokens for use in our models. Secure tokens have become common in modern web applications. 
You might use them for: 

* Invitations
* User registrations, 
* Resetting lost user passwords,
* Token-based authentication.

In the past, everyone had to craft their own custom solution for this, Rails 5 wants to make the process a little easier and more secure by providing a standard way to create and manage secure tokens.

There are two parts to the process. The first is getting the columns set up on your database, and you can do that either from the command line, using rails generate or by writing a migration.

###### Rails generate command
```
rails g model Invitation auth_token:token
```

###### Migration class
```ruby
class CreateInvitations < ActiveRecord::Migration
    def change
        create_table :invitation do |t|
            t.token :auth_token
        end
    end
end
```

In both cases, you can see that I'm providing the column name, which is gonna be my token name, and I made that auth_token, and then the type of column is going to be token. 
That's a new type that's introduced in Ruby on Rails 5, so instead of having a string, or an integer, or a date time column, we tell it that it's a token column.

Once we have our database set up and migrated then in our model we can add the method `has_secure_token`and provide the name of our token. 

```ruby
class Invitation < ActiveRecord::Base
    has_secure_token(:auth_token)
end
```

If we don't provide any argument, it defaults to token, once we've got that, our model will now automatically generate secure tokens for auth token.

```ruby
invite = Invitation.new
invite.save
invite.auth_token # => "05oplsfvvkwXI3q2349Oains"
```
We create a new invite and we save it to the database, if we ask that object for auth token, you can see that it will automatically have created this random secure string for us. 


If we wanna refresh that token to something new, then we can call `regenerate_auth_token`: 

```ruby
invite.regenerate_auth_token # => true
```

And it will regenerate that token and give us a new random string:

```ruby
invite.auth_token # => "hnXBv7R2XValsgwga1atTrIX"
```
Notice that that's a bit of a dynamic method. `regenerate_auth_token is based` on the name of my token, if my token had been named something like invite code, well then the name of the method would be `regenerate_invite_code`, so it's `regenerate_` followed by the name of your token and that would regenerate it for you, so these secure tokens that it generates are gonna be random, 24-character tokens.

Under the hood, at least at the moment, it uses secure random base 58. That could certainly change in the future. If that wasn't deemed to be appropriate, but at the moment 24-characters using base 58, because of the size of the token, and the number of characters, collisions and race conditions are possible but they're unlikely. 

To prevent this, you're encouraged to use a unique index on the database column to ensure that the value is always unique, now the Rails migration, using that token type automatically adds that unique index for you, but if you create your own migration, or if you're using an existing string column, then you'll wanna make sure that you add your own unique index to that column.

It's an unlikely scenario, but if you add it then you'll never need to worry, and that's all there is to being able to use secure tokens in your ActiveRecord models.

You can find out more about the ActiveRecord Secure Tokens at the your Rails documentation [page](http://api.rubyonrails.org/classes/ActiveRecord/SecureToken/ClassMethods.html).


### Abort ActiveRecord Callbacks

Another improvement in Ruby on Rails 5, is that the procedure for aborting ActiveRecord callbacks has changed. In Rails 5, callbacks no longer halt whenever the last statement in the callback is false. Instead you must explicitly stop them, by using throw abort.

Let me demonstrate the issue that this change is meant to solve. 

Here's a class for product, in Rails 4. 

```ruby
# Rails 4
class Product < ActiveRecord::Base
    before_save :set_in_stock_value
    before_save :process_product_image

    def set_in_stock_value
        self.in_stock = Inventory.in_stock_for(code)
    end

    def process_product_image
        # Some image processing code
    end
end
```

Notice that it has two before save callbacks. Set in stock value, and process product image.

They're going to be executed in the order they're defined, when set in stock value is called, it sets in stock value to the returned value of inventory in stock for, and this value's going to be true or false. Here's the problem, when a Ruby method finishes, if there's no explicitly returned value, then it returns the last value evaluated inside the method. That's just how Ruby works, in this case, that will be the value returned by inventory in stock for, which could be false, and if it's false, then set in stock value returns false.

When the before save callback receives that false signal, in Rails 4, it halts the callback chain and it aborts the save, so the second callback, process product image never gets called, and the record isn't saved to the database and not because we wanted it to fail, but just because of a quirk in our code. In Rails 4, this was an easy mistake to make without meaning to. In Rails 5, this code would work as expected, and inadvertently returning false will not stop the whole show. 

Instead, in Rails 5, you must explicitly tell the callback to stop.

```ruby
# Rails 5
class Product < ApplicationRecord

    before_save :set_in_stock_value
    before_save :process_product_image

    def set_in_stock_value
        throw(:abort) unless Inventory.in_stock_for(code)
        self.in_stock = true
    end

    def process_product_image
        # Some image processing code
    end
end
```
In the above example, you can see I've rearranged the logic of set in stock value so that it aborts the save if inventory in stock for, returns false. Otherwise, it sets the value to true, now you might not want to do this, this is in fact a different behavior than what we had on the last slide.

What I wanna show you is that you must explicitly use throw abort anytime you do want the callback chain to stop, so if you have code in the existing application that was depending on returning false to stop the callback chain, then you'll wanna swap those in with using throw abort instead.

This is a configurable option. If you create a new Ruby on Rails 5 project it adds an initializer called callback terminator. 

```ruby
# config/initializers/callback_terminator.rb
ActiveSupport.halt_callback_chains_on_return_false = false
```

Where you can set the value for halt callback chains on return false, equal to true or false. A new project's gonna set it to false. If you're upgrading from an older version of Rails, then you might wanna add this initializer as well. If you were to add the configuration and then change it to true, you'd get the Rails 4 behavior again. However, you should only use it like that, long enough for you to upgrade you application to the new behavior.

Eventually, once everyone's had an opportunity to make the change, I expect this configuration is likely to go away.

### Changes to parameters/params

Ruby on Rails 5 has made a few improvements to the strong parameters code that we use in our controllers, you're going to want to beware of any code which treats the parameters as if it was a simple hash, because it's not going to be anymore. It's no longer going to inherit from [HashWithIndiferentAccess](http://api.rubyonrails.org/classes/ActiveSupport/HashWithIndifferentAccess.html), as a fundamental class in Rails that allows us to access a hash, by providing either symbols or strings as the keys of the hash. Instead, our params are now going to be inheriting from a standalone parameters class.

It's still going to have indifferent access, we can still access the values by using either symbols or strings, and also using the square bracket notation that you're used to. 

```ruby
params[:page] = 1

params[:page]
# => 1

params['page']
# => 1

```

It's not a hash with indifferent access anymore. It's an object which has similar behaviors, let me show you a way in which it's not the same. 

In Rails 4, you might call slice on your params, in order to extract some of the key value pairs you wanted to work with, so `params.slice``and then sort by and sort dir, will return back the key value pairs, and the values that correspond to them.

```ruby
# Rails 4
params.slice(:sort_by, :sort_dir)
# => {:sort_by => 'name', :sort_dir => 'asc'}
```

In Ruby on Rails 5, calling the same code, might return nothing to us:

```ruby
# Rails 5
params.slice(:sort_by, :sort_dir)
# => {}
```

Even if those values exist in the params, and the reason why, is because params has been made more secure. It's stronger now, we'd already started that with strong params, it's carried over to the way that we interact with params inside our controllers as well, if we call `.class` on our params, we'll see the `ActionController::Parameters` class.


```ruby
# params is now a class, not a hash
params.class
# => ActionController::Parameters
```

In Ruby on Rails 5, if we wanna get access to that hash, then we have to either tell the params to convert it into a hash, using `to_unsafe_h`, and then slice those values

```ruby
# Rails 5
params.to_unsafe_h.slice(:sort_by, :sort_dir)
# => {:sort_by => 'name', :sort_dir => 'asc'}
```

Or we will have to permit the values and then we can work with them:

```ruby
# Rails 5, better technique
params.permit([:sort_by, :sort_dir])
params.slice(:sort_by, :sort_dir)
# => {:sort_by => 'name', :sort_dir => 'asc'}
```

Permit is the same way that we normally work with strong parameters, so if we had already previously permitted them, for that purpose, then they'll be permitted here, but we need to make sure that they've been permitted.

If they're not permitted, we don't have access to them, the only other way is to use to `unsafe_h` and that's meant to sound scary. It's meant to let you know that you're leaving behind the security features that params has and you're working with a basic hash from then on.

The example above is with `#slice`, but it's also true for `#except`, `#extract`, `#select`, `#reject`, `#merge`, `#transform_keys` and all of the exclamation point versions of those (eg. `#slice!`).

They only work with permitted params, so you either need to permit the params first, or you need to convert the parameters object into an unsafe hash. 

There's another place that it makes a difference When you're comparing params against a hash. 

```ruby
# in Rails 4
hash = {:sort_by => 'name', :sort_dir => 'asc'}
if(params == hash) { ... }
```

Before, you would just be comparing params against a hash, and you'd be comparing two hashes to each other.

But now, you're gonna be comparing an object against a hash, so the correct way to do that, is either take the hash and turn it into a similar parameters object or take the parameters and turn it into the unsafe hash, using the technique we saw earlier.

```ruby
# in Rails 5
hash = {:sort_by => 'name', :sort_dir => 'asc'}
obj = ActionController::Parameters#new(hash)
if(params == obj) { ... }
```

The older version still works in 5.0, but you'll get a deprecation warning that starting in 5.1 you'll be required to do it the new way, so to summarize, in most cases, parameters still act like a hash, but we have to be careful now because there are some cases when they don't act just like a hash and these security features come into play. 

It's a small, but important change, that could break some of your older code.


### Enumerable#without

Ruby on Rails 5 adds a new method to Enumerable called without. Enumerable#without returns a copy of the enumerable, but without the elements that we've passed in as arguments.

Now, the enumerable is actually a module that's included in several other classes, such as array, and hash, so what we're really talking about is being able to use this without method on an array or a hash. 

So for example let's say we have an array, like fruit. In that array we have apple, plum, banana, and orange. 

```ruby
fruit %w(apple plum banana orange)
```

We can call ``fruit.without` plum and orange and we'll get back an array that has apple and banana in it.

```ruby
fruit.without('plum', 'orange')
#=> ['apple', 'banana']
```

If we have a hash which has keys for a, b, c and d and then various numbers corresponding to those.

```ruby
hash = {:a => 27, :b => 34, :c => 56, :d => 4} 
```
then if we call a hash without b and c, we would get back a hash that contains the key value pairs for a and d.

```ruby
hash.without(:b, :c)
#=> {:a => 27, :d => 4}
```

This method is mostly just to give us a nicer way to do something we've already been able to do before. 

For example, you could do it with reject:

```ruby
users.reject {|u| paid_users.include?(u)}
```

Or if you have two arrays, you could do it using subtraction, so I could have:

```ruby
users - paid_users

users - [current_user]
```

But it can be a bit of hassle when you have only a single user because then you have to make an array in order to get the subtraction to work. 

Using without is cleaner and it clearly states what we're trying to achieve and as a bonus it works with arrays, with a list of arguments and with a single item. Without is a nice addition. It's easy to use and it makes your code clearer.

```ruby
users.without(paid_users)

users.without(current_user)

```

For more information about `Enumerable#without` see the Ruby Enumerable#without [documentation](http://api.rubyonrails.org/classes/Enumerable.html#method-i-without).


### Enumerable#pluck

The new method on enumerable called pluck is similar to an existing [method](http://api.rubyonrails.org/classes/ActiveRecord/Calculations.html#method-i-pluck) on ActiveRecord called pluck. 

The way that works, is if we have a model, like user, we can ask it to pluck ID and the username and it will fire a new SQL query and pull back just those values, just the ID and the username and that's true, even if we've already previously loaded this scope, it goes directly to the database in order to ask for those values. Now often, we want to write methods which operate either on arrays or scopes.

They don't really care which one it is, they want to do the same thing on both of them, by adding this enumerable pluck method, it allows enumerables to pluck values in the same way that active record pluck does.

Let see an example:


Let's say that I have an array of users and inside that array I have three different hashes and inside the hashes, I have an ID and a username for each of the users.


```ruby
users = [
    {:id => 21, :username => 'rails'},
    {:id => 23, :username => 'django'},
    {:id => 41, :username => 'nodejs'}
]
```

If I call `users.pluck(:username)`, I'll get back an array that contains just those usernames. 

```ruby
users.pluck(:username)
# => ['rails', 'django', 'nodejs']
```

It will go through the array, it will iterate through each item and it will return back to me the value that corresponds to the key that I've given it.

If I ask it for two different keys, then it will give me an array of arrays.

```ruby
users.pluck(:username)
# => [[21, 'rails'], [23, 'django'], [41, 'nodejs']]
```

You can see the first item in the array is 21, rails. That's the ID and the username, now where this comes in useful, is that in Rails 4, if we had users, now if we get a scope back, we get all our users, but then we wanted to also pluck the username from those users, we had to use pluck on ActiveRecord.

```ruby
# in Rails 4
users = User.all
# Sends SQL : SELECT * FROM users
```

And that would send new SQL and that was the only way to do it, even though we already had all these users loaded, if we wanted to use pluck, we had to go use ActiveRecord and send another query to the database.

```ruby
# in Rails 4
User.pluck(:username)
# Sends SQL : SELECT username FROM users

# => ['rails', 'django', 'nodejs']
```

The other choice was to go through the users and do something like map, in order to get all these values back.

In Rails 5, we don't have to go back to the database again. If we already have a scope loaded, so, for example, we have `users = User.all`. 

```ruby
# in Rails 5
users = User.all
# Sends SQL : SELECT * FROM users
```

Well now, we can call pluck on that enumerable, on that array of objects and we can ask it for username.

```ruby
# in Rails 5
User.pluck(:username)
# No additional query needed!

# => ['rails', 'django', 'nodejs']
```

And it doesn't fire off another SQL query, it does what we would expect, it goes into the users, it iterates through them and it returns the values that we're looking for. 

So now, we can call pluck on either an ActiveRecord object or on an array or a hash and it doesn't make any difference.

It still returns back the values that we're looking for.

For more information about `Enumerable#pluck` see the Ruby Enumerable#pluck [documentation](http://api.rubyonrails.org/classes/Enumerable.html#method-i-pluck).


### ArrayInquirer


One of the major improvements the Rails 5 is the addition of the class ArrayInquirer. 
ArrayInquirer provides a convenient syntax for checking the contents of an array, it works a lot like StringInquirer. 

You may not have worked with StringInquirer before, so let me first demonstrate how that works.

Let's say that we have a string, such as active. We can take that string and we can pass it as an argument to `ActiveSupport StringInquirer.new` to get back a StringInquirer object. 

```ruby
# String Inquire example
status = ActiveSupport::StringInquirer.new('active')
```

That object is going to work just like a string in almost every single case

```ruby
put status      # => 'active'
```

But it's gonna have one additional ability, which is that we can acquire about its contents and we can do that by using:

```ruby
status.active?     # => true
```

The StringInquirer will take whatever value we've passed in before the question mark and compare it against the value of the string, so it returns true because active is in fact the value of the string. 

```ruby
status.pending?     # => false
```

If I call status.pending?, it returns false, because the value of the string is not pending, so it's a little bit of a magic method name. Anything that would be put before that question mark is going to be compared against the value of the string.

Rails actually uses this for `Rails.env`, so we have the ability to call` Rails.env.production?` and it returns true or false, depending on whether the value of the string set for the Rails environment is production or not, but it still works just like a string in every other way.

```ruby
Rails.env.production?   # => true
puts Rails.env          # => 'production'
```

So now let's look at [ArrayInquirer](http://api.rubyonrails.org/classes/ActiveSupport/ArrayInquirer.html). 
Let's say that I just have a basic array and in that array I have three values: producer, director and actor.

```ruby
# ArrayInquirer example
array = [:producer, :director, :actor]
```

I can turn this into an ArrayInquirer object in two different ways.

The first, is that I can use that ActiveSupport ArrayInquirer new technique that we just saw with StringInquirer.

```ruby
roles = ActiveSupport::ArrayInquirer.new(array)
roles.class # => ActiveSupport::ArrayInquirer
```


I take the array and I pass it in as an argument to ArrayInquirer new and then I set that to roles and I get back that ArrayInquirer object.

Another easier, I think, way to do this, is to just call `.inquiry` on the array and it will return that ArrayInquirer object for you, It's a little shorter. 

```ruby
roles = array.inquiry
roles.class # => ActiveSupport::ArrayInquirer
```

In both cases, we get back an object that works like an array, except that it has these extra inquirer abilities added to it.

So if I have roles, producer, director, actor and I add `.inquiry` to the end:

```ruby
roles = [:producer, :director, :actor].inquiry
```
Now I have the ability to inquire about the contents of that array.

```ruby
roles.producer? # => true
roles.director? # => true
roles.actor? # => true
roles.writer? # => false
```

`roles.producer?` returns true, so does `director?` and `actor?`, `writer?` returns false because writer is not inside the array. 

In addition to these single queries. 
We also have the ability to use any on it as well, so I can ask whether any of the values are equal to director or writer and I can do that using either symbols or using strings, it doesn't matter.

```ruby
roles.any(:director, :writer) # => true
roles.any('director', 'writer') # => true
```
Both cases it returns true, because director is one of those values even though writer is not.

If I were to ask it though, if roles contains writer or special effects. 

```ruby
roles.any(:writer, :special_effects) # => false
```
It will returns false, because neither one of those values are inside my array.

You're not going to want to do this to every single one of your arrays, but there are certain cases when it is gonna be convenient to be able to have a way to easily inquire about the contents of an array and in those cases this is gonna be a great tool to have.

For more information about ArrayInquirer see the Ruby `ArrayInquirer`[documentation](http://api.rubyonrails.org/classes/ActiveSupport/ArrayInquirer.html).



### Per-Form CSRF tokens

Ruby on Rails already provides developers with easy to use tokens to guard against Cross-Site Request Forgery, or CSRF for short.

In Ruby on Rails 5, these have been further strengthened using per-form CSRF tokens. Per-form CSRF tokens prevents a particular technique of form hijacking, it isn't easy to steal a CSRF token, but it is possible and a stolen CSRF token could permit fake forms to still be sent to the server and be considered as legitimate.

By providing a unique CSRF token per-form, then a hacker who steals a token can only send that same form back to the application.CSRF tokens are only valid for the method and action for which they were generated.

A newly created Ruby on Rails 5 app will add a file to the initializers which will set per-form CSRF tokens to be true, with the following code:

```ruby
config.action_controller.per_form_csrf_tokens
```

For upgraded applications or others apps which do not have that initializer in place, it's going to be set to false by default.

It will keep the old behavior until you explicitly configure it to be true. So you can add that file to your initializers, or you can add this configuration to ActionController, or any controller, `per_form_csrf_tokens = true` and then you'll be using per-form tokens instead of using a more general token.

It's an easy change to make, there's no reason that you shouldn't do it, and it increases the overall security of your Rails application.

### belongs_to Requires Parent

There's an important change to the default settings for `belongs_to` in Ruby on Rails 5.

Typically, when we work with `belongs_to` we would expect to save that child record to the database after we've already saved the parent record to the database, That's usually the way it works.

Starting in Ruby on Rails 5, that's going to be a requirement. The `belongs_to` is going to be required to have a parent by default, that's going to be a value that's controlled by a setting `belongs_to_required_by_default`.

A newly generated Rails 5 app is going to have an initializer that sets this value to true. 

Older apps are going to keep the false setting by default if they don't have that configuration set, if you're upgrading, you'll need to opt in to the new behavior by providing that configuration in the initializers. If you opt into it, then what will happen is that anytime you add a `belongs_to` to a class, it's going to automatically add `validates_present_of`, the parent for you, you don't need to add it yourself, it's going to automatically do that for you.

You used to be able to do that yourself by using the required option when you declared the `belongs_to`. Instead, because it's going to be `:required` by default, you should use the `:optional` option whenever it's not required by default. 

So you should stop using `:required` and let the default behavior kick in, you should use `:optional` if you don't want it to be the case. 

In the Rails 5 as I said, in most cases I think you're going to save the parent object before you save the child object because you're going to want that parent's ID to be able to relate it to the child.

The one place I think this might cause a lot of pain for people is going to be in your tests because there may be times when in your test cases you're just saving an object to the database without a parent because you're just doing unit tests or something like that on it. You're going to need to be careful about those because now you're going to have that `validates_presence_of` `:parent` that can potentially keep the object from saving if it doesn't also have a parent.



## Deprecations and Deletions

Ruby on Rails 5 has many new features and improvements. But it's also going to remove a few features as well.

### Deprecated Filter Callbacks

Previously in your controllers you could use `before_filter`, `after_filter` and `around_filter`. 

Instead, now you want to use `before_action`, `after_action` and `around_action`. That's the accepted way to do these, so stop using the filter version and use the action version instead. That applies also to the variations on these, such as `append_before_filter`, `prepend_before_filter` and `skip_before_filter`. All of them have equivalents using the word action instead.
 
So anywhere in your controllers where you have before filter, you want to change that to be before action instead.

```ruby
class ProductsController < ApllicationController
    #before_filter :confirm_logged_in   => DEPRECATED
    before_action :confirm_logged_in #  => TO BE USED NOW
    def index
        @products = Product.all
    end
end
```


 The new methods have been with us for a little while. They were introduced in Rails 4.2, but now the old ones are being deprecated and has been removed entirely in Rails 5.1. 
 
 It's an easy change to make, so there's no reason not to update them all today.

 ### Deprecated Render Nothing

Another change in Ruby on Rails five is that render nothing has been deprecated.

```ruby
class ProductsController < ApllicationController
  
    def index
        if authorized_user
            @products = Product.all
            render('index')
        else
            render(:nothing => true)    
        end
    end
end
```

The above example is what I'm talking about. It was possible before in your controller to simply render nothing at times. So, for example, if we have an authorized user we would render a page to them, but otherwise, we might render nothing, I did that with `render(:nothing => true)`.

When you do it this way, it's not actually rendering nothing. It's still sending an HTML header with the status code letting the browser know that the request was answered successfully.

That's the effect that we probably want, but it isn't entirely clear from what you see in the code that that's what's happening and therefore, it's been decided that it's much better to use the head method, which returns an HTML header with whatever status code or options that you set, but which has an empty body.

```ruby
class ProductsController < ApllicationController
  
    def index
        if authorized_user
            @products = Product.all
            render('index')
        else
            head(:ok) #  => TO BE USED NOW
        end
    end
end
```

They do the same thing, but the second one makes it clear what's happening. For this reason, using `render(:nothing => true)` in Rails 5 will still work, but you'll also get a deprecation warning in your logs or in the console and if you are using the Rails 5.1 version this won't work anymore. 


### Moved RecordTagHelper to [Gem](https://github.com/rails/record_tag_helper)

The RecordTagHelper class has been moved to RubyGem. RecordTagHelper is made up, really of just two methods, those are `div_for` and `content_tag_for`.

```ruby
div_for(@person, :class => 'student')

content_tag_for(:tr, @person) {...}
```

So those are no longer going to be available. What those did, was they took an object, like an Active Record object and they constructed a div or a content tag based on the attributes of that object. 

It's a feature that not a lot of people were using and so it was deemed better off in a Ruby gem than it is inside the Rails core.

If you still want to use it, you can use the Ruby gem. All you have to do is add `record_tag_helper` to your gem file, run bundle install and then you'll have that same functionality available to you. But for everyone else, these methods are going to be removed.

I do want to make a footnote here, which is that the `content_tag` method is still available, that's different than `content_tag_for`. `content_tag_for` was specifically about working with the attributes of an active record object and `content_tag` just helps you generate a basic content tag and it's still on the Rails core.