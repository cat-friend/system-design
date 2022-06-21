# Next steps
High-level trade-offs:
* Performance vs scalability
* Latency vs throughput
* Availability vs consistency

Everything is a trade-off!

# Performance vs scalability
Link: https://github.com/donnemartin/system-design-primer#performance-vs-scalability

**scalable** means a service that results in increased performance in a manner that is proportional to resources added; in general, increasing performance means service more units of work, but it can also mean to handle _larger_ units of work (ex: when datasets grow)

Examples:
* performance problem - system is slow for a single user
* scalability problem - system is fast for a single user but slow under heavy load

# Latency vs Throughtput
* Latency - time to perform an action or produce some result
* Throughput - number of such actions or results per unit of time

Aim to have maximal throughput with acceptable latency

# Availability vs Consistency

## CAP theorem
In a distributed computer system (like the cloud), you can only choose two:
1. consistency - every read receives the most recent write or an error
2. availability - every request receives a response, no guarantee that it will contain the most recent version of the information
3. partition tolerance - system continues to operate despite arbitrary partitioning due to network failures

Since networks aren't 100% reliable, _must_ choose partition tolerance and then make a software tradeoff between consistency and availability.

### CP - consistency and partition tolerance
Every read receives the most recent write or an error & system continues to operate despite arbitrary partitioning due to network failures

Good choice if your business needs require atomic reads and writes

### AP - availability and partition tolerance
Every request receives a response, no guarantee tat it will contain the most recent version of the information & system continues to operate despite arbitrary partioning due to network failures

Responses return most readily available version of data available on any node; writes take some time to propagate when the partition is resolved

Good choice if business needs allow for eventual consistency or when the system needs to continue working despite external errors

# Consistency patterns
Why is consistency important? Have multiple copies of the same data - need to synchronize them so clients have a consistent view of the data

## Weak consistency
* After a write, reads may or not see it. Best effort approach is takent.
* used in memcached
* works well in real-time use cases: VoIP, video chat, realtime multiplayer games
    * ex: if you're on a phone call and lose reception, when you reconnect, you don't hear what was spoken during connection loss

## Eventual consistency
* After a write, reads will eventually (ms) see it; data is replicated asynchronously
* Seen in DNS and email
* works well in highly available systems

## Strong consistency
* after a write, reads will see it; data is replicated synchronously
* works well in systems that need transacctions, like file systems and RDBMSes

# Availability patterns
Two patterns support high availability:
1. fail-over
2. replication

## Fail-over
### Active-passive
Heartbeats (packets) are sent between the active and passive server on standby; if the heartbeat is interrupted, the passive server takes over the active's IP address and resumes service

Downtime length determined by passive server status - running in 'hot' standby or needs to spin up from 'cold' standby

### Active-acctive
Both servers are managing traffic, spreading the load between them

Public-facing servers - DNS would need to know about the public IPs of both servers

Internal-facing servers - application logic needs to know about both servers

### Disadvantages
* fail-over adds more hardware, additional complexity
* potential for loss of data if the active system fails before any newly written data can be replicated dto the passive

## Replication
Discussed more in the database section

## Availability in numbers
* availability - often quantified by uptime (or downtime) as a percentage of time the service is available.
    * generally measured in `9`s:
        * 99.9% is 3 9s
        * 99.99% is 4 9s

### Availability in parallel vs in sequence
If a service consists of multiple components prone to failure, the service's overall availability depends on whether the components are in sequence or in parallel
* in sequence
    * overall availability _decreases_ when two components with availability < 100% are in sequence
``` total availability = Xavailability * Yavailability```

    * ex: 99.9% * 99.9% - 99.8% availability

* in parallel
    * overall availability _increases_ when two components with availability < 100% are in parallel:
``` Availability (Total) = 1 - (1 - Availability (Foo)) * (1 - Availability (Bar)) ```

# Domain Name System (DNS)
A DNS translates a domain name to an IP address+
DNS is hierarchical
Router/IISP provides information about which DNS server(s) to contact whendoing a lookup
Lower level DNS servers cache mappings - mappings could become stale due to DNS propagation delays
DNS results can also by cached by browser or OS
Cache lifecycle is determined by time to live (TTL)

Relevant terms:
* name server (NS) record - specifies the DNS servers for your domain/subdomain
* mail exchange (MX) record - specifies the mail servers for accepting messages
* record (address) - paints a name to an IP address
* CNAME (canonical) - ponits a name to another name or `CNAME` ex: example.com to www.example.com or to an `A` record

Types of traffic routing:
* weighted round robin
    * prevents traffic from going to servers under maintenance
    * balance between varying cluster sizes
    * A/B testing
* latency based
* geolocation based

## Disadvantages
* accessing a DNS takes time --> introduces slight delay; delay can be mitigated by caching
* DNS server management could be complex, generally managed by governments, ISPs, and large companies
* DNS services can be attacked; prevents users from accessing websites without knowing the website's IP address(es)

# Content delivery network
## Push CDNs
## Pull CDNs

# Load balancer
## Active-passive
## Active-active
## Layer 4 load balancing
## Layer 7 load balancing
## Horizontal scaling

# Reverse proxy (web server)
## Load balancer vs reverse proxy

# Application layer
## Microservices
## Service discovery
# Database
## Relational database management system (RDBMS)
### Master-slave replication
### Master-master replication
### Federation
### Sharding
### Denormalization
### SQL tuning
## NoSQL
### Key-value store
### Document store
### Wide column store
### Graph Database
## SQL or NoSQL
# Cache
## Client caching
## CDN caching
## Web server caching
## Database caching
## Application caching
## Caching at the database query level
## Caching at the object level
## When to update the cache
### Cache-aside
### Write-through
### Write-behind (write-back)
### Refresh-ahead
# Asynchronism
## Message queues
## Task queues
## Back pressure
# Communication
## Transmission control protocol (TCP)
## User datagram protocol (UDP)
## Remote procedure call (RPC)
## Representational state transfer (REST)
# Security
