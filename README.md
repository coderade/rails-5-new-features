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