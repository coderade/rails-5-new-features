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