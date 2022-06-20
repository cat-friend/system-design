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
