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
* CDN - globally distributed network of proxy servers
    * serves content from locations closer to the user
    * usually, static files (HTML/CSS/JS, photos, videos) served via CDN
    * dynamic content - Amazon's CloudFront
    * website's DNS resolution will tell clientns which server to contact

* Why is CDN good?
    * users receive content from data centers close to them (low latency)
    * servers do not have to serve requests that the CDN fulfills

## Push CDNs
* Push CDNs receive new content whenever changes occur on the server
* you take full responsibility for providing content, uploading directly to the CDN and rewriting URLs to point to the CDN
* can configure when content expires and when it is updated
* content is uploaded only when it is new or changed, minimizing traffic, but maximizing storage

* optimal uses - sites with a small amount of traffic, sites with content that isn't updated often as content is placed on the CDN once, instead of being re-pulled at regular intervals

## Pull CDNs
* Pull CDNs grab new content from server when the nfirst user requests the ncontent
* leave the content on server and rewire URLs to point to the CDN
* ** SLOWER REQUEST ** until the content is cached on the CDN

* TTL determines how long content is cached on CDN
* CDN storage space minimized but can create redundant traffic if files expire and are pulled before they have actually changed

* Sites with heavy traffic work well with pull CDNs - traffic is spread out more evenly with only recently-requested content remaining on the CDN
    * probably uses an LRU cache

## Disadvantages
* CDN costs depend on traffic, so could be significant --> should be weighed against additional costs you would icur not using a CDN
* content might be stale if it is updated before the TTL expires it
* CDNs require changing URLs for static content to point to the CDN

# Load balancer
* Definition - load balancers distribute incoming client requests to computer resources such as application servers and databases
* load balancer returns the response from the computer resource to the appropriate client
* route traffic to a set of servers serving the same function
* **Load balancers are good at**
    * preventing requests from going to unhealthy servers
    * preventing overloading resources (servers)
    * helping to eliminate a single point of failure

* can be implemented via hardware (expensive) or with software (ex: HAProxy)

* **Benefits**
    * SSL termination - decrypts incoming requests/encrypts server responses so backend servers do not have to perform these potentially expensive operations
        * removes the need to install X.509 certs on each server
    * session persistence - issue cookies and route a specific client's requests to same instance if the web apps do not keep track of sessions

* fail-protection
    * set up multiple load balances, either in active-passive or active-active mode

* methods of routing
    * random
    * least loaded
    * session/cookies - user connected to which server last?
    * round robin or weighted round robin
    * Layer 4
    * Layer 7

## Layer 4 load balancing
* decides how to distribute requests by looking at information within the transport layer
* typical information **FROM HEADER** used:  source, destination IP address, ports
    * does NOT consider packet contents
* forwards network packets to and from the upstream server, performing Network Address Translation (NAT)

## Layer 7 load balancing
* decides how to distribute requests by looking at information in the application layer
* typical information used:  contents of the header, emssage, and cookies
* L7LB terminate network traffic, reads the message, makes a load-balancing decision, then opens a connection to the selected server
    * example: direct video traffic to video hosting servers, directs sensitive user billing traffic to security-hardened servers


## Tradeoffs
**L4LB** - requires less time and computing resources
**L7LB** - requires more time and computing resources, but performance impact can be minimal on modern commodity hardware

## Disadvantages
* LB can become a performance bottleneck if it does not have enough resources or if it not configured properly
* introducing a load balancer to help eliminate a single point of failure results in increased complexity
* a single load balancer is a single point of failure; multiple load balancers -> increases complexity

## Horizontal scaling
* LBs can help with horizontal scaling -> improving performance and availability
    * scaling via commodity machines is more cost efficient and results in higher availability than scaling up a single server on more expensive hardware
        * easier to hire talent for commodity hardware than for specialized enterprise systems

### Disadvantages of horizontal scaling
* introduces complexity and requires cloning servers
    * servers should be stateless - should not contain any user-related data like sessions or profile pictures
    * sessions can be stored in a centralized data store such as a database or a persistent cache

# Reverse proxy (web server)
* Definition - a web server that centralizes internal services and provides unified interfaces to the public
* Method:
    * requests from clients are forwarded to a server than can fulfill it before the reverse proxy returns the server's response to the client
* Benefits
    * increased security - hide information about backend servers, blacklist IPs, limit number of connections per client
    * increased scalability and flexibility - clients only see the reverse proxy's IP, allowing you to scale servers or change their configuration
    * SSL termination - decrypt incoming requests and encrypt server responses so backend servers do not have to perform these potentially expensive operations
        * removes the need to intall X.509 certs on each server
    * compression - compress server responses
    * caching - return the response for cached requests
    * static content - serve static content directly

## Disadvantages
* introducing a RP results in increased complexity
* a single RP is a single point of failure, having multiple RPs further increases complexity

## Load balancer vs reverse proxy
* **LB** is usedful when you have multiple servers
* **RP** is useful even with only one server/application server because of benefits listed above
* NGINX and HAProxy can support L7RP and load balancing

# Application layer
* separate web layer from application layer -> allows you to scale and configure bboth layers independently
* adding new API results in adding application servers without necessarily adding additional web servers
* SRP advocates for small and autonomous services that work together
* small teams with small services can plan more aggressiely for rapid growth
* workers in the napplication layer also help enable asynchronism

## Microservices
* definition - suite of independently deployable, small, modular services; each service runs a unique process and communicates through a well-defined, lightweight mechanism to serve a business goal

## Service discovery
* software can help services find each other by keeping track of registered names, addresses, and ports
* health checks help verify service integrity and are often done using an HTTP endpoint

## Disadvantages
* adding an application layer with loosely coupled services requires a different approach from an architectural, operations, and process viewpoint (vs a monolithic system)
* microservices can add complexity in terms of deployments and operations

# Database
## Relational database management system (RDBMS)
* definition - collection of daata items organized in tables
* must have ACID properties
* many ways to scale

### ACID
Atomicity, Consistency, Isolation, Durability
* Atomicity - each transaction is all or nothing
* Consistency - any transaction will bring the database from one valid state to another
* Isolation - executing transactions concurrently has the nsame results as if the transactions were executed serially
* Durability - once a transaction has been committed, it will remain so

There are many techniques to scale a relational database: master-slave replication, master-master replication, federation, sharding, denormalization, and SQL tuning.

### Master-slave replication (what a gross name)
Main - serves reads and writes, replicates writes to 1+ mirrors
Mirrors can also replicate to additional mirrors in a tree-like fashion
If main fails, system can operate in read-only mode until a mirror promotes to main or a new main is provisioned

### Master-master replication
both mains serve reads and writes and coordinate with each other on writes; if either goes down, the system can continue to operate with both reads and writes

* **Disadvantages**:
    * need a load balancer or need to make changes to application logic to determine where to write
    * most main-main systems are either loosely consistent (violates ACID!!) or have increased write latency due to synchronization
    * conflict resolution comes more into play as more write nodes are added and as latency increases

### Disadvantages of replication
* additional logic is needed to promote mirror to main
* potential loss of data if main fails before any newly written data can be replicated to other nodes
* writes replayed to the replicas -- if there are a lot of writes, the read replicas can be overburdened with replaying writes and can't serve as many reads
* the more read mirrors, the more you have to replicate, leads to greater replication lag
* on some systems, writing to the master can spawn multiple threads tow rite in parallel, whereas read replicas onl support writing sequentially with a single thread
* replication adds more hardware and additional complexity

### Federation
* definition - aka functional partitioning - splits up databases by function
    * ex: products db : users db : forums db
* result is less read and write traffic to each database and therefore less replication lag
* smaller databases result in more data that can fit in memory, which in turn results in more cache hits due to improved cache locality
* no single central main serializing writes, you can write in parallel, increasing throughput

#### Disadvantages
* NOT EFFECTIVE if your schema requires huge functions or tables
* need to update application logic to determine which database to read and write
* joining data from two databases is more complex with a server link
* adds more hardware --> additional complexity

### Sharding
* definition - distributes data across different databases such that each database  can only manage a subset of data
    * ex:  facebook - users separated into different databases by last name
* common ways to shard - by last name or by geographic location

#### Benefits
* less read and write traffic, less replication, more cache hits
* index size reduced - improves performance with faster queries
* no single central main serializing writes, allows to write in parallel with increased throughput

#### Disadvantages
* need to upate application logic to work with shards, could result in complex SQL queries
* data distribution can become lopsided in shards
    * ex: "gonzales" is a common last name
    * rebalancing adds additional complexity
    * sharding function based on consistent hashing can reduce the amount of transferred data
* joining data from multiple shards is more complex
* sharding adds more hardware and additional complexity

### Normalization
* multistep process to eliminate data redundancy and enhance data integrity in the table; sets the data into tabular form and removes the duplicated data from the relational tables
* Normal Forms - the process of taking a database design and applying a set of formal criteria and rules
* goals of database normalization - stramline data by reducing redundant data
* drawbacks of redundancy:
    1. data maintenance becomes tedious - data deletion and data updates become problematic
    2. creates data inconsistences
    3. insert, update, and delete anomalies become frequent - an update anomaly means that the versions of the same record, duplicated in different places in the database, will all need to be updated to keep the record consistent
    4. redundant data inflates the size of a database and takes up an inordinate amount of space on disk

### Denormalization
* attempts to improve read performance at the expense of some write performance - good for read-heavy systems
* redundant copies of the data are written in multiple tables to avoid expensive joins
* useful in federation and sharding scenarios - data are distributed, managing joins across data centers further increases complexity; denormalization might circumvent the need for such complex joins

#### Disadvantages
* data is duplicated - requires more disk space
* constraints can help redundant copies of information stay in sync -> increases complexity of the database design
* denormalized database under heavy write load might perform worse than its normalized counterpart


### SQL tuning
* benchmark - simulate high-load situations
* profile - enable tools such as the slow query log to help track performance issues

Benchmarking and profiling might point you to the following optimizations:
* Tighten up the schema
    * MySQL dumps to disk in contiguous blocks for fast access
    * Use `CHAR` instead of `VARCHAR` for fixed-length fields
        * `CHAR` - allows for fast, random access
        * `VARCHAR` - must find the end of the string before moving onto the next one
    * Use `TEXT` for large blocks of text such as blog posts; `TEXT` allows for Boolean searches; `TEXT` results in storing a pointer on disk that is used to locate the text block
    * Use `INT` for larger numbers up to 2^32 or 4 billion.
    * Use `DECIMAL` for currency to avoid floating point representation errors
    * Avoid storing large `BLOBS` and instead store the location of where to get the object instead
    * `VARCHAR(255)` is the largest number of characters that can be counted in an 8-bit number, often maximizing the use of a byte in some RDBMS
    * Set the `NOT NULL` constraint where applicable to improve search performance
* Use good indices
    * columns that are queried could be faster with indices
    * indices are usually represented as a self-balancing B-tree that keeps data sorted and allows searches, sequential access, insertions, and deletions in logarithmic time
    * placing an index can keep the data in memory, requiring more space
    * writes could also be slower since the index also needs to be updated
    * when loading large amounts of data, it might be faster to disable indices, load the data, then rebuild the indices
* avoid expensive joins
    * denomalize where performance demands it
* partition tables
    * break up a table by putting hot spots in a separate table to help keep it in memory
* tune the query cache
    * in some cases, the query cache could lead to performance issues

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
