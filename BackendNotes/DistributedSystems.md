# Distributed Systems
Lecture by Ned Ruggeri. How do you deploy your application to cloud such that it doesn't crash while serving a large user base?

## Web Application Process
- **Amazon EC2**: rent computers in Amazon data center
  - Heroku does this for you
  - Can run make apps, run servers (ex.: instead of localhost3000, [username]3000), etc. remotely on this computer
  - Public DNS, IP, etc.
- Difference between domain name and IP
  - House address -> **Internet Protocol address**
  - Resident name -> **domain name**
  - Phonebook in the cloud -> **Domain Name Service**


1. Phonebook (Domain Name System DNS) -> lookup address (IP) using name (domain name)
2. HTTP request to address
3. Server, server hands request to application
4. Application communicates with backend database (Postgres and mySQL have own server that accepts database requests, typically only from same machine as app)

## Scaling
- ^ doesn't scale well: not enough CPU power to compute all the requests
- Fixes:
  - **Vertical scaling**: pay to improve computing power of current machine
  - **Horizontal scaling**: add more servers to handle more requests


### Scaling Up: Database (RAM and Disk)
- **Hard disk**: used to be very slow, was a rotating physical device like a record player/cassette
- **RAM**: created to speed things up, circuit using electric signal communication, but wasn't persistent
- **Solid state**: hybrid of the two, circuit, persists, but not has fast as RAM


- Normally database is what causes slow application
- Database uses hard disk and RAM
  - Copies data from disk to RAM (RAM essentially used like a cache for disk data) -> fast GET request
    - Run out of space: buy more RAM, optimize active set (most popular FB posts, StackOverflow questions, etc.) to be on RAM
  - Writes data to disk and RAM -> persistent
    - Extra work, but usually much more read requests than writes

### Scaling Out: App Machine & Database
- 2 options: scale out work of 1. App or 2. Database
  - 1 is easier

Multiple App Machines
- Rent more AWS machines (collectively the **application tier**), one solely for the database, one is Load Balancer
  - User -> DNS -> IP, HTTP request -> load balancer -> app -> backend -> back
- **Load balancer**: doesn't respond to user request, configured to copy headers of HTTP request and pass to machines w/ app -> later copies response back to user
  - Failover: if one of app machines crash, load balancer doesn't promptly receive response -> excludes that machine from work distribution

Multiple Databases
- More databases (collectively the **database tier**), one is "leader," rest are "followers"
  - All rails apps must send write requests to leader db -> leader copies new data over to followers
    - **Synchronous replication**: copies data to followers after each new write
    - **Asynchronous replication**: batch processing, better performance (less taxing, one big job at a time is preferred by computers, can decide ordering to be efficient)
  - Read requests to any db (disadvantage: followers may not have updated writes)
  - Multiple leaders: no advantage, bc 2+ leaders still need to transfer write info to each other, conflicts can occur
- Promotion: if leader fails, one follower promoted


- **Sharding/partitioning**: each obj is entirely handled by 1 machine determined by cat_id % num_databases, no need for inter-machine communication
  - Ex.: cat_24 % 6 -> Shard4
  - If Shard4 fails, there can be follower of Shard4 to act as backup
  - Con's: JOIN queries require interaction with many machines (solve by denormalizing data), resizing is awful
- **Denormalize**: don't normalize (ex.: hold reference to cat in friends table with cat_id), instead have copies of info in diff locations, bc joins queries are now not good
  - Ex.: ask for all friends of one cat, improve scalability by having all data in one place
    - Curie's friend Breakfast's data in Shard4 where Curie is, even though Breakfast is in Shard6
  - Tradeoffs: copying -> more storage, updating complicated (but improvements to read > detriment to write)
    - Thus, use for when reads >> writes

# ACID Database
- A, C, I, D properties:
  - **Atomicity** and transactions
    - **Transaction**: SQL query either completes fully, or rollbacks
  - **Consistency**: db constraints enforced (not null, uniqueness, all ensured)
  - **Isolation**: two transactions that don't conflict can be done in parallel, but two that do conflict
    - Ways to achieve
      - **Serial execution**: transactions alw processed one at a time, slow
      - **Parallel execution**: ideal, must understand when there are conflicts
  - **Durability**: saved data never lost


-------------- RLee's Overview
# Architecture

## Standard Web App Architecture
- Browser --> load balancer --> app servers --> caching layer --> CDN(s) --> DB

### Databases (slow)
- Relational DB: easy to use and query, denormalization often used to scale and avoid joins
  - Scaling
    - High read load + master-slave: leader handles writes, followers read
      - Ex.: Twitter, FB
    - High write load + shards: not for relational DB, sharding (designated handlers for each obj)
      - Ex.: WhatsApp


- NoSQL: schema not enforced
  - Document-based: ex. MongoDB (giant hash map w keys and vals as JSON obj)
  - K-V stores: ex. Redis (like document-based, but less powerful bc no indexes or joins)
  - Distributed dbs: used by all big websites, ex. Cassandra (cluster of dbs --> allows failures)

- Failover: prevent single point of failure
  - Leader and followers (master-slave relationship)

### To Make a Site Faster: CDN and Caching

#### Content Delivery Network (CDN)
  - CDN: system of distributed servers that deliver content to users, based on user, webpage, and server location
  - HTTP flow w a CDN
    - Browser makes DNS request for domain name that's handled by a CDN --> server handling DNS requests looks at incoming request to determine best set of servers to handle

#### Caching Layer
- Caches: generally stored in memory (like RAM), are K-V stores
  - Store common queries (ex.: someone's tweets), expensive queries (ex.: trending hashtag), static data
  - Cache invalidation: if cached data becomes invalid or unnecessary
    - Ex.: LRU cache

### Application Server


## Sources:
https://github.com/rlee0525/TechnicalConceptsForInterviews/blob/master/Architecture.md
