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
### AP - availability and partition tolerance

# Consistency patterns
## Weak consistency
## Eventual consistency
## Strong consistency

# Availability patterns
## Fail-over
## Replication
## Availability in numbers

# Domain Name System (DNS)

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
