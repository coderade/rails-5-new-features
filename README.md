# Rails 5 new features

This project show the new features, improvements and other important changes to Ruby on Rails version 5. It will shows how to use the most important features of Rails 5, including action cable, action controller renderer, turbolinks, the active records, attributes API, and how to use Rails exclusively as a Json API. And will discuss too the many other smaller changes in improvements such as the Rails command router, new date and time methods, secure tokens, and changes to parameters.

In addition, it will show the features that are being deprecated or completely removed. This is an important topic if you have an existing project that uses any of these features. 


## About Ruby on Rails 5

* First major release in three years - That was for the version four, and it's been about a year and a half since Rails 4.2 came out.
* Faster
* Less memory usage
* Less time doing garbage collection (GC)
* Requires 2.2.2 or greater
* [GC of symbols (Incremental GC)](https://www.sitepoint.com/symbol-gc-ruby-2-2/)
* **Incremental GC: ~2x faster** - Previously, that meant that Ruby would periodically go through all the objects in memory, mark the ones that were still fresh or being used, and then make a second pass to remove the unmarked objects from memory.
 The problem is that that GC sweep through memory takes some time, it can slow down your code while it's happening. Ruby 2.1 improved the speed of this process somewhat by classifying objects in memory based on the likelihood they would need to be garbage-collected. But a large collection could still slow things down. Well, now, in Ruby 2.2, we get incremental GC. And the basic idea is to break up the scanning and removal of objects so that the process happens incrementally, not all in one big sweep. So while the total work and the total time may still be the same, it's broken up among many smaller operations.
* **Optimizes common operations** - The core team has optimized common operations, the code has fewer dependencies on other code in libraries, and there are fewer object allocations to memory. Now, that may seem like a small point that doesn't make much difference, but, as one example, link and URL generation in Ruby on Rails 5 is 44 percent faster.
* Fewer dependencies
* Fewer object allocations
* **Development environment is faster** - The development environment uses the Puma web server now, instead of using WEBrick, and development mode used to check the modification time of all of your files in your project to know if anything had changed, so that it could automatically reload the development environment for you. But now, there is a file system monitor, which notifies Rails whenever something changes. There is no more requirement to check the modification times of all the files. And that makes development feel snappier when you are working. And on top of these performance improvements.


## Major features

### Action Cable and Websockets

* Framework for working with Websockets.
* Allows real-time features using a constant connection between server and clients.
* Comprised of server-side Ruby and client-side Javascript
* Uses Redis pubsub to track communications

#### Uses for Action Cable

* Streaming content such as video
* Interactive content such as online games, or any similar application where you're swapping mini messages per second between the client and the server
* Realtime communications such as chat

#### Differences between the HTTP Model and the Action Cable 

##### HTTP Model

* Data exchanged is independent of connection (each time a communication needs to happen, a new connection is created)
* Wrapped data: request headers + data
* Headers can 1-2KB large (even if the data is tiny) - includes things like cookies, that the browser has stored. Even if the data is tiny, it still has to wrap it in these headers, and the server overhead goes both ways.
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
* HTTP request-response cycle can serve more clients - So when I make a request for a webpage. The server sends me the webpage, and then while I'm reading the webpage and deciding what I want my next decision to be, my next request to the server, it can go ahead and serve other clients in the meantime.
* No page caching - Different of the HTTP requests, with something like a webpage, that can be cached and sent very quickly back to the client. With WebSockets we have to dynamically respond all the time, and in fact, most applications just simply
* Most applications do not require realtime communication


With WebSockets we have to dynamically respond all the time, and in fact, most applications just simply do not require real-time communication, so ActionCable and WebSockets is a great solution for people who need real-time communication, but if you don't need it then you're probably better going off with a more traditional HTTP model.

#### Using ActionCable

##### ActionCable Terminology

* Cable -  single connection between a server and a client That's one single client, so when we open a connection between them, we call that a cable.
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

Next, we're gonna need to create our channel, and a new Rails 5 app should have a directory for channels where you can put this code, and you're gonna create channels in much the same way that you normally create controllers, so while they're not exactly the same, you'll find that they are similar, so here you can see I've got a class defined for `ChatRoomChannel` that inherits from `ApplicationCable Channel`, and then you can see that I have a subscribed action, and a speak action.

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

There will usually be an unsubscribed action as well. Subscribed and Unsubscribed are the two standard actions. These two actions do whatever code you want to do whenever a user subscribes or unsubscribes from a channel. Often what you wanna do when they're subscribing is to start streaming data to them across the channel, and you can see that that's what I'm doing in my subscribe method when I'm calling stream from. Now, speak is a custom action of my own creation. Here I've added just a basic example. When the speak action is called, then the ActionCable server is going to broadcast a message on the chat room channel, and you can see that I'm passing in whatever I want that data to be.

Anyone that's streaming from that channel anyone who's subscribed, will be sent that message immediately. Once we have our chat room channel set up, and we have subscribed, unsubscribed and any of the custom actions that we wanna have inside there. 

Then we're ready to set up the JavaScript side of things. In your JavaScript, you'll need to set up the cable like so. Here you see I have variable for App, and then I'm setting App.cable equal to `ActionCable.createConsumer`. This gives our JavaScript object all the features that it needs to communicate with the server.

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

Once we have it set up, we can use it to set up a room on our app object, which creates a new subscription to the chat room channel. Here you can see I'm setting `App.room` equal to `App.cable` subscriptions create, so I'm creating a new subscription and I'm assigning it to room. I'm gonna call it `ChatRoomChannel`, and inside you'll see that I'm defining a number of functions. I've got functions for connected, disconnected, received, and speak.

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
**connected**, **disconnected** and **received** are standard functions in `ActionCable` that you'll probably always want to have. Whenever I'm connected, the JavaScript in connected function will execute. Whenever I'm disconnected, the JavaScript in the disconnected function will execute, and the received will gets called anytime data is received from this channel. 

In the above example, in my received function. I'm just calling a simple JavaScript alert to display whatever message is received. You could instead replace data on the page, or add the data to some existing content. For example, if this was a real chat room, you'd probably want to append the message received to the end of the things that already been said in the chat room.

Now, speak is a custom function of our own creation. It can be called anything we like, but notice that what it does, is it calls perform speak. This is how we call the controller action speak that we created earlier. We send it data, and that's the message that we wanna speak. Remember, that action just broadcast our message to anyone who subscribed to this same channel.

Now that we've defined all of our Ruby actions on the server side, and all of our JavaScript functions on the client side. 

In order to use it, we just have JavaScript call our speak function, like so, so we call `App.room.speak`, anywhere in our code. We provided the message, and then it goes through the code that we want. 

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

Let's review what the above code looks like. It calls this perform speak, and it passes along our message That then in turn calls `ChatRoomChannel` speak. Which in turn calls `ActionCable` server broadcast, and that sends our message out to that channel, and anyone who subscribed to it is going to get it.

It's gonna be streamed to that WebSocket. Once that's received by the client, it's gonna trigger `App.room.received` on all the subscribers. Which in turn calls alert and alerts with our message, so that we see what the message was.

Now this is a very simple example. Meant to just give you the flow of how things work. There's a lot more that you can do, and a lot of configurations to help you to do them. If you wanna experiment or learn more, a great resource is the rails actionCable examples that are on [github](https://github.com/rails/actioncable-examples) and official Action Cable [documentation](http://guides.rubyonrails.org/action_cable_overview.html).



### ActionController::Renderer

* Render templates independently of controllers.
* Useful for ActionCable

#### Overview

Another major feature of Ruby on Rails 5, is ActionController Renderer. ActionController Renderer, allows us to render templates, but to do it independently of our controllers. It's particularly useful for things like ActionCable. We can render bits of content, that we broadcast to all our subscribers, and we don't have to load up all of the action controller code, with parts that we don't need. Like cookies and redirects. We can just use the rendering feature on its own.

#### Examples

The way that you use it, is very straight forward. It's just like you would render inside a controller, so instead of just calling render, we now call `ApplicationController.render`, and then tell render what we wanna render.

On the bellow sample a template called products/index will be rendered, and it's gonna look for that template in its normal place. It's gonna look for it in the views directory, inside the products directory. It's gonna look for a template called index.

```ruby
ApplicationController.render(
    :template => 'products/index'
)
```    

You can also use the shorthand form for this, that you're probably more familiar with, and that is just to call ApplicationController.render, and then the template name that we want.

```ruby
ApplicationController.render('products/index')
```

Another nice shorthand, is that if instead of calling render on ApplicationController, we call it on one of our custom controllers, which inherits from ApplicationController. Then we don't even have to provide the directory for where to find the template. It knows that it's going to look in the products directory, because we're working with the ProductsController, so that's where it's going to look, by default, for this index template, and just like our controllers, we don't have to just render html.

```ruby
ProductsController.render('index')
```

We can render json, for example. If we wanted to provide json back as a snippet, to action cable, we can do that just like this:

```ruby
ApplicationController.render(
    :json => Product.all
)
```

The other possibilities for things you can render are templates, actions, partials, file, inline, plain, text, html, json, js, and xml.

Just like you're used to having in your controllers. You can pass local variables to the templates. Just like you normally would. Here you see an example where I'm rendering a partial, for products_product, and I'm providing local variable values for product and for user. 

```ruby
ApplicationController.render(
    :partials => `products/_product`,
    :locals => {
        :product => Product.first,
        :user => current_user
    }
)
``` 

There's only one thing that's slightly tricky about this, and that is, that in your controller, it's common practice to set up instance variables, which are then automatically going to be bound to the ERB template that we render, so that they're available for use inside that template.

That binding doesn't happen automatically with renderer, but render lets us pass instance variables for binding to the template, using assigns, so on the below example, you can see that I'm passing in both locals and assigns. This would mean that inside that partial, I would be expecting product to be a local variable, and user would be an instance variable.

```ruby
ApplicationController.render(
    :partials => `products/_product`,
    :locals => { :product => Product.first },
    :assigns => { :user => @user }
)
``` 

In case the difference between assigns and locals is not completely clear, here's another example that shows them both being applied to the rendering of a bit of inline ERB, so my template is that inline ERB.

You can see that in the first blank I have an instance variable for version, and a local variable then in the second blank for adjective, so I'm using assigns to set the instance variable. I'm using locals to set the local variable. 


```ruby
(
    
    :inline => "I think Ruby on Rails version <% @version %> is <% adjective %>!",

    :assigns => { :adjective => 'sweet' }
    :locals => { :version => 5 },
)
``` 
Overall, rendering works pretty much the same way that it does inside our controllers. What's different in Rails 5, is the fact that we've gained the ability to do this rendering outside of controllers. We don't need a controller in order to render code. We can do it inside our channels.

We could do it inside our jobs, our mailers, inside our rake tasks. There's all sorts of places that rendering can now take place without having to have all that controller code loaded in.

For more information about Renderers see the Ruby `ActionController::Renderer`[documentation](http://api.rubyonrails.org/classes/ActionController/Renderer.html).

### Rails as JSON API

* Based on Rails::API [gem](https://github.com/rails-api/rails-api)
* API for clients to access data and application logic
* Backend for client-side frameworks (Client eg.: JS, Flash, Iphone, or Rails)

This feature almost made it into Rails 4 back in 2012, but instead it became a stand alone Ruby gem, which has now been incorporated into Rails 5.

You should note that it is possible to have an existing Rails application which doubles as a JSON API. In other words it serves up both HTML and JSON. But that's not what this example is. This will talk about making a Rails app which is exclusively an API. We're gonna be setting up our application from the start so that it omits a lot of the normal Rails application code.

#### Why Rails as JSON API?

* No need for templates, HTML, CSS and JavaScript - there's no need for templates, HTML, CSS, and JavaScript, all that can be left out, and when we use our generators in Rails it's gonna skip generating the views, the helpers, and the assets whenever we create a new resource.
* Most application logic is outside the View layer - most of the application logic is performed inside our controllers and models and not inside the view. So we can let the client just simply be our view layer for us
* Less controller and middleware code - when we go to create a new resource, the Rails as an API is going to leave out modules which are primarily useful for browser applications, like cookie support. It won't put jQuery or turbolinks into our gem file because it knows we won't be using them.
* Configured for API-style applications
* Familiar Ruby framework, RubyGems, Bundler
* Sensible defaults and generators
* Keep key features: security, routing, caching, error handling, logging and etc.
* Development and test environments -The test is an important aspect too, so we can continue to write test for our application in a way that we're used to do our Rails apps.


#### How do we use it?

To create a Rails API application, all you need to do when you create a new Rails app is to add the dash dash api option to the end. As the following command:

```
rails new my_backend_api --api
```

The biggest difference is this single line. inside config/application.rb there's a new line that says config.api_only = true. And that's what tells Rails to be in API mode in everything that it does. Every time you use a generator, every time you do anything in your application, it's going to be in API mode. If you're converting over from an existing application, then this is a line that you'll need to add.

```ruby
# config/application.rb
module MyBackendApi
    class Application < Rails::Application

        config.api_only = true    
    end    
end    
``` 

Another difference is in our controllers. We still have controllers, but we don't want all the HTTP parts that we aren't going to be using. So application controller is going to inherit from the class action controller API. 

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::API

end    
  
``` 

Normally it would inherit from action controller base. This change leaves out code like cookies and sessions, which we don't need, but still gives us other parts of the controllers that we do want. Such as before actions, security methods, page caching. 

And then the controllers that we generate are going to inherit from application controller, which in turn inherits from the API.

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

So our controllers are going to work the same way, and the generated content of our controllers will also be different. Notice in the above example that these controllers rendered JSON resources by default. They also include create, update, and destroy actions. But unlike regular controllers they don't automatically have a new and edit action. Because typically those are just HTML pages for submitting forms. And an API client would be expected to take care of that business for us. 

Our routes still use resourceful routes and we define them the same way, but if you take this and then run rake routes on it you'll see that the new and edit routes that are normally present when doing restful routing are no longer included with the API version.

```ruby
# config/routes.rb
Rails.application.routes.draw do

    resources :todos
    
end    
```

We don't need them the same way that we don't need the actions that correspond to them. You can add them back if you decide you do. But they're not there by default. With Ruby on Rails 5, Rails developers who want to create backend applications where Rails serves as a JSON API will find it much easier than they have in the past. And it will also omit a lot of the application code that you don't need to make your application as lean and fast as possible.

For more information of how use Rails for API-only Applications, see the official [documentation](http://edgeguides.rubyonrails.org/api_app.html).


### Turbolinks 5

Turbolinks was first introduced in Ruby on Rails 4. This version of Turbolinks has now been renamed as [Turbolinks Classic](https://github.com/turbolinks/turbolinks-classic), and version five, which comes with Ruby on Rails 5, is a ground up rewrite with a new flow and new events, but which still uses the same core idea as Turbolinks Classic.
The new version is faster than the old one, and also offers the ability to do partial replacements of webpages. We'll take a look at that feature in a moment.

#### Why Turbolinks?

* Performance of a single-page application - that's part of what JavaScript frameworks like to boast about
* No client-side JavaScript framework required - you can work directly with HTML pages in your existing Rails application. 
* Links do not load completely new pages
* Fetch new content with Ajax; swap `<body>`; merge `<head>` - it fetches the page using asynchronous JavaScript, also known as Ajax, and then it takes the returned page and it replaces the body of the current page with the new body, and it merges the content of the page header with the existing page header. 
* CSS and JS is retained (does not need to be reloaded or re-rendered)
* Content is still full HTML page, not fragment or JSON
* Current browser scroll position is maintained
* Still changes browser's URL and history - the back button will work as expected
* Much faster, can feel almost instantaneous (<1 second)


#### Using Turbolinks

* To use it, we need first to add the turbolinks to Gemfile: `gem 'turbolinks'`
* Run the `bundle install` command
* Add to JS manifest: `//= require turbolinks`

You don't have to use Turbolinks all the time, you can have some links that opt out of it. 

```html
<a href="/some_url" data-turbolinks="false"> 
    This link will not use Turbolinks
</a>
```

As the above example, there I have a link that I've marked with `data-turbolinks=false`, and this now becomes a link that will not use Turbolinks when it loads. Otherwise, all your other links will automatically use Turbolinks, and you'll get that speed boost as a result.

There is one gotcha when working with Turbolinks, and that is that it's very common for people to write JavaScript that either activates on loading the window or when the document is considered ready, and that's not going to be true with Turbolinks anymore, because our document and our window are already loaded.

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

All we're doing is swapping data into the existing document, so if you find that your JavaScript isn't firing anymore, then instead you wanna use the EventListener, turbolinks, load, and if you'd listen for turbolinks load, your JavaScript will fire again.

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

The first version of Turbolinks simply replaced everything within the body when a link was clicked. That kept the HTML head, CSS and JavaScript, but the bulk of the page was still reloading, and you can continue to use Turbolinks in that way, and you'll get a big speed boost by not reloading that extra stuff, but the new version of Turbolinks that's in Ruby on Rails 5 has an additional feature.

You can reload only parts of a page by marking content as either permanent, or as temporary. 

Look the below example. Let's imagine we have a page, and inside the body I have three divs. One with an ID of nav, one with an ID of flash, and one with an ID of comments. You can see that I've marked the nav as being data turbolinks permanent. That means that this div is not going to change after the initial load by default. Then I've got div id flash marked data turbolinks temporary and that's going to be updated with every request unless I specifically ask it to stick around, and then I've got a normal div which I've just called comments, and that's typically going to change on each page load.

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

For the first example, if I just do a simple render. Let's say I render comments/index, it's going to keep the nav div around. It's not going to make any changes to it. It will leave it in place because I've marked it permanent. It's gonna update all the other parts of the page.

In the second example, I've specifically targeted comments by using change. You can also use append and prepend in the same way, so I'm targeting comments, therefore it's going to update the flash and the comments.
It updates the comments because I targeted it. It updates the flash because I marked that it's being temporary, so it's still going to be refreshed each and every time, but the nav and all other parts of the page. Any other content on the page is left alone. 

Like the third example, if I instead tell it to keep the flash, well then the flash will stay around, so now in the third example you'll see that it's gonna keep the nav, because I've marked it permanent. It's going to keep the flash, because I've explicitly asked it to, but all other parts of the page are gonna be updated.

On the Last example, you can see that I'm updating everything including the nav by using flush true. This is still using Turbolinks. It's not opting out of Turbolinks, it's just telling it that all the parts of the page should be replaced. The entire body gets replaced in that case.

The below example show hoW the things are done inside your controller. You can also do the same thing in your JavaScript. This is essentially what's happening under the hood. When Ajax makes this call. That is that we're calling `Turbolinks.visit` comments, and then we can provide any of those options we just saw.

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

Change, keep, flush, as well we can use append and prepend, and those do the exact same thing. If you wanna find out more about Turbolinks. Learn more about some of the configuration options that are available to you. You can find out more about it at the turbolinks official [repository](https://github.com/turbolinks/turbolinks).


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

* First, the argument that you pass  Must be an `ActiveRecord::Relation` (query or scope) - That's why I had to use post dot where on the above example, again, because we needed to created a `ActiveRecord::Relation`
* The arguments must be the same model - So if my exterior query has to do with post, then the thing that goes inside or should also be regarding Post. 
* And the arguments must differ only by their `#where` or their `#having` clause - that's because that's what's going on under the hood. `ActiveRecord::Relation#or` is taking any where or having clause that we pass in on a scope, and it's appending them to the existing where and having clause using or.
* Should not use `#limit`, `#offset`, or `#distinct` - you can still use limit, offset, and distinct on the exterior scope, but not on the relation that's being passed into or.

If you keep these guidelines in mind, I think that you'll find that or provides a handy tool to our Active Relation query toolbox.