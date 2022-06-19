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
