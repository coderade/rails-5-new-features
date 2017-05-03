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