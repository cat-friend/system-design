# Scalability Lecture at Harvard
Link: https://www.youtube.com/watch?v=-W9F__D3oY4

* Topics covered:
    * Vertical scaling
    * Horizontal scaling
    * Caching
    * Load balancing
    * Database replication
    * Database partitioning

## Vertical scaling
### Intro
Minimal features to expect from webhosting company:
* not blocked in country
* SFTP vs FTP
    * secure - encrypted traffic
        * not a big deal for files that are meant to be downloaded, but import for usernames and pw
        * FTP exposes UN and PW
* resources shared?

### VPSes
Virtual private server
What's the difference? **you get your own copy of the OS**
Take a super fast server and chop it up into the illusion of multiple servers, like hypervisor
Multiple virtual machines on a single machine
Sys admin - might have access to VM and files
Costs more than a shared server

AWS EC2 - elastic compute cloud - good for unplanned, unexpected growth

### Actual information
Stressing system - CPU, running low on disk space, almost at max RAM usage --> what's the easiest, most obvious solution? get more CPU/disk space/RAM --> throw money at the problem; however, there's a limit to how much vertical scaling that you can do
* can only buy a machine that's so fast
* can only support so much RAM
* can only throw so much money at something or reach technological limit

Most servers have at most 2 CPUs, cpu has more cores - ex. quad core -> computer can do 4 things at once
PATA/IDE - old tech - 7500 RPM
SATA - what's in desktops - 7500 RPM
SAS drive - serial attached scuzzy - faster 15k RPM
If website has db - db writes to disk - if you touch disk a whole lot, put a SAS drive
SSD - no moving parts, perform faster than mechanical drive

## Horizontal scaling
Accepting that there are limitations so architect system so you don't hit the ceiling; use cheaper hardware and more of them
What does this actually mean?
* If you have a whole bunch of servers instead of just one, need to distribute request over the server
    * load balancing! - traffic coming from traffic is balanced across servers - not one server handles all the requests
    * instead of DNS (returns IP address of server 1) can return IP address of load balancer, then load balancer determines route
        * BE servers can have private addresses
            * private vs public IP addresses
                * private - can't see - privacy properties - can't be contacted directly
                * public - the world can see

## Load balancing
### How can load balancer decide which server to handle the request?
Request arrives at load balancer -> balancer decides which server to send request (usu least busy) -> request sent to Server X (TCP/IP) -> server gets packet -> server responds to request -> response goes to load balancer -> load balancer sends response
* all servers must be identical

What's a simpler approach?
* round robin - multiple IP addresses for a host, load balancer acts as a DNS and returns a different IP address for each server - don't need bi-communication
    * caveats
        * one server could be burdened with a huge task -> RR still sends user to that server
        * caching - browser and OS caches responses of common lookups
        * TTL - time to live for answer from a DNS server - if you are a power user, might be a few minutes/hours/days before you get assigned to another server
        * session is saved via cookie on the server but if by RR, you get sent to another server, you will need to log in again or if you have a cart, your items in the cart are saved across a server
            * Dedicated servers for specific task/data
                * pros - can eliminate this session-based  problem
                * cons - can overload the dedicated servers for that task
                * cons - no redundancy - problem for uptime
* factorization - session data saved on a server
    * cons: weakness in network topography - affects uptime if the session server dies
* RAID - redudundant array of independent disks
    * ex: RAID0, RAID1, RAID5, etc
    * RAID0 - assume multiple hard drives, stripe data across them - write chunks of data across multiple drives - doubles speed of writing
    * RAID1 - mirror data across them - stop data in both places - performance overhead but faster to access and there's redundancy
    * RAID10 - 4 drives - striping and redundancy
    * RAID5, RAID6 - variants of RAID1 3, 4, 5 drives - only 1 used for redundancy - better economy of scale
    * RAID6 - any 2 drives can die before you have to buy a new drive

### How do you implement load balancing?
* Software
    * Elastic load balancer
    * HAProxy
    * Linus virtual server
* Hardware
    * Barracuda, cisco, etc - very overpriced
### Sticky sessions
visit a website multiple times, you have the same session object (end up on the same BE server)
* shared storage
* cookies
    * store session data privacy issues, size issues
    * can store token to access specific server - exposes server IP/ID
        * what if IP changes?
        * security risk
        * load balancer can have a key:value pair, key: random num, value: actual IP of server
* PHP accelerators - limits discarding of PHP off-codes; .pyc is python equivalent

## Caching
Caching is bad if value has changed
### Craigslist example
file-based caching approach
redundancy in every single page
better performance for serving static content, but sacrifices disk space
trivial changes are a big deal - need to find and replace/regenerate all bazillion places

### MySQL query cache
My.cnf - config file
query_cache_type = 1 -> enables query cache
If you enable query cache

### memcache
* expensive to do SELECT * FROM Users - execute once and save results in RAM and cache in RAM
* store in table that has indexes
* cache is finite, disk is finite - eventually cache can get so big that you can't keep it on the machine
    * solution - LRU cache

### archive tables
* compressed - slower to access but take up less disk space
* typical usage - log files, dx data

## Database replication
Make automatic copies of something
### Single main database paradigm
* Have a main database - read/write from and to; main has multiple mirrors - their purpose is to get copies of the main db; main and mirrors are all identical
* pros
    * uptime
    * good for more read-heavy websites
* cons
    * what if the main drive dies?
        * single point of failure for writes

### Double main database paradigm
* Two mains are mirrors
* still need to route traffic

### Multi-tiered architecture
Client network --> load balancer
load balancer --> multiple webservers receive read/write queries --> go to load balancer --> go to MySQL mirrors

Points of failure - MySQL main, load balancers

load balancers sometimes use heartbeats - sent horizontally; two modes:  active-active or active-passive

## Database partitioning
Ex:  Facebook server for each school
Problem - need to cross Harvard-MIT boundary
Name cluster-based load balancers

### Summary of info up to this point
1. Have multiple web servers - need sticky sessions
    1. What's the best option?
        * Load balancer, load balancer has cookie to remember which server to send user to
2. database
    * need redundancy
3. partition database - sends user to same database each time based on user data
    * problem - single point of failure
    * solution - 2 main databases that mirror each other
    * need cross connect - but now load balancer needs to be done in code
        * solution - connect load balancer between web servers and databases
            * this load balancer can't do application-level load balancing, only binary load balancing; application-level load balancing (ex: user last names) will be done on the web servers as that level operates via HTTP requests
            * cons - single point of failure
            * solution to LB - redundant load balancer
    * then need redundant switches; switches need to be intelligent so that there aren't loops
    * point of failure - data center goes offline (ISP, power to building)
        * solution - redundant data center inside and between geographical availability zones

Multiple identical data centers, how to load balance?
* DNS can do geography based load balancing
* still have potential downtime - browser/computer has cached IP address of that data center - need to wait for TTL to expire

What kind of internet traffic needs to be allowed in and out of the building?
* in: TCP.80, 443, 22 (ssh, ssl vpn)
* load balancers -> web servers
    * TCP 80
    * offload SSL to load balancer - pros: put SSL certificate only on load balancer, can get cheaper web servers
* web servers -> databases
    * SQL queries - TCP 3306

If you have firewalling capabilities, can further lock things down - only expose certain ports because principle of least privilege - only open doors that people need to go through - limits bad actors
