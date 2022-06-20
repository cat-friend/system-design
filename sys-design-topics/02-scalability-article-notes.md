# Scalability for Dummies

Link:  https://www.lecloud.net/tagged/scalability/chrono


## 1. Clones
Every server contains exactly the same codebase and does not store any user-related data, like sessions or profile pictures, on local disc or memory -> then it doesn't matter which server the user is connected to

Sessions - store in centralized data store which is accessible to all application servers
* can be external db or an external persistent cache, like Redis
    * external persistent cache will have bbetter performance than an external database
** `external` - data store does not reside on the application servers but in or near the data center of application servers **

What about deployent? How can you make sure that a code change is sent to all servers without one server still serving old code? Capistrano.

outsourcing sessions and serving the same codebase from all servers -> can now create an image file from one of these servers
the image file is like a "super clone" that all the new instances are based upon; whenever you start a new instance/clone, do an initial deployment of latest code

## 2. Database
Slow database?
1. Path 1 - MySQL - main and mirror database servers; upgrade master server with increasingly more RAM --> will eventually hit a tech ceiling

2. Path 2 - denormalize right from the beginning and don't include joins in any database query
    * can stay with MySQL and use it like a NoSQL database or can switch to a better and easier to scale NoSQL database like MongoDB or CouchDB.
    * joins will be done in the application code
    * will eventually need to introduce a cache

## 3. Cache
in-memory caches like Memcached or Redis
--> file-based caching makes cloning and auto-scaling a PITA

cache is a simple key-value store and it should reside as a buffering layer between the napplication and the data storage

### Data caching patterns
1. Cached database queries
* Most commonly used caching pattern
* Whenever db is queries, store the result dataset in cache
* hashed version of query is the cache key
* next time run query, check if it's already in the cache

Cons
* expiration
    * hard to delete a cached result when you cache a complex query
    * when one piece of data changes, you need to delete all cached queries that may include that now-changed piece of data

2. Cached objects
* see data as an object like you already do in code (classes, instances, etc)
* let class assemble a dataset from database and store the ncomplete instance of the nclass or the assembled dataset in the cache

Pros:
* makes asynchronous processing possible - application consumes the latest cached object and nearly never touches the database

Common objects to cache:
* user sessions
* fully rendered blog articles
* activity streams
* user <-> friend relationships

## 4. Asynchronism
1. "bake the bread at night and sell them in the morning" - do the time-consuming work in advance and serve the nfinished work with a low request time
    * used to turn dynamic content into static content

2. queue of jobs for worker to process - when finished, frontend waits for "job is done" signal; asynchronism is great  because backends become nearly infinitely scalable and frontends become snappy

** if you do something time consuming, try to do it always asynchronously ** 
