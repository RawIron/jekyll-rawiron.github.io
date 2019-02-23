---
layout: post
title:  "Building services that support 10_000's of users"
date:   2019-02-22 14:39:12 +0200
categories: devops
---


## Design Considerations

### Do the math
Start with some very simple estimates.

Given a distribution that 60% of all the users play in a 4 hour window you have 60K players in 4 hours. Now let's assume each player on average sends 100 game requests we have 6M game requests in 14400sec. Given an even distribution it is around 400 game requests/sec. Furthermore each game request on average makes 20 requests to the data service and we get 8,000 queries/sec.

### Predictable response times
For example each service has to respond for 99% of all requests with less than X ms.

The core loop must be really fast. To achieve this keep data in memory. Memory is the most precious resource, so make sure the working set is small. Also avoid to optimize for peak performance but instead go for a stable constant performance.

### Non-blocking
Any blocking call in the code within the core loop can bring down the application at some point. This usually happens when calling an external service (curl call) or a non-critical service (logging). 

For example the event-logging service. After a game request (e.g. buyGameItem) is completed a log event is sent to the log service (hosted on a cluster of servers). At some point the log service will slow down and after some time the servers hosting the game services get flooded by waiting processes and so causing the game services fail to respond in time.

### The smaller and the more the better
It should be possible to clone the service and add more servers hosting the service. When a service runs on many servers then a single server failure will be noticed only by a small amount of players. For example for 8 servers only 12.5 percent of all players might notice the failure.

It should be possible to host a different kind of services on their dedicated servers. This will make troubleshooting on production much easier. Also it makes it easier to scale different parts of an application independently like static content delivery, writes to persistent storage, posting virals, etc. 

### Profile and Monitor enabled
Load tests cannot replace the real production load. The design should make sure that the service can be profiled on production.
In a php project the xhprof extension could be used.


## Performance, scalability, availability
High performance does not mean it scales. And an application that scales does not have to be high performance. Also you can scale almost any application if cost are ignored. 

For example you could scale a very slow service by deploying the service over a large amount of servers and than load balance requests over those servers. As an example you get 10K requests per second but the service can handle only 10 requests per second. In that case you would deploy 1K servers to handle the load.

### The overall stack is important
A very fast data layer (which is expensive) can compensate for poorly performing game service implementations.
The services never cache any data after they were read from the data service. Instead every time the state-less service needs state information a request is sent to a data service. In some cases the identical data is transferred from the data service 10-20 times per game service request. And this means that you can reduce the number of request to the data service by a factor of 10-20 just by cleaning the code in the game service implementations.

### Leave head room
Always add the “extra” resources.
A very simple example is that in case of failure the load will be balanced over less resources and taking into account that the peek load is much higher than the average load the “extra” resources are really important to keep the services available. And it is likely that something fails under high load. This could be dealt with more elegantly when your services are hosted in an elastic scaling cloud. 

### Data grow
Test with large data sets. Lots of performance issues do not show while the data sets are very small.
A _quest_ system which launches with a few _quests_ and a few players log their progress to the _quest_ service. Over time more and more quests will be added to the service and _quest_ completion and quest progress data will grow.

Persistent data which keeps growing will waste precious in-memory cache space.
The _quest_ progress service which does not remove progress data after _quest_ completion.

### Availability
Available often means the services should respond if the application is not in maintenance mode. In contrast to “Telco availability” which requires all maintenance can be done without downtime.

In most projects availability is taken care of in a very pragmatic way. Usually a manual procedure to replace a failed server. Given the size of a project and the resources at-hand this is often a good decision.

               
## Load Testing and Optimization
Maximum throughput is not what you want. Peek performance is not what you want. Average response time is not what you want. Really important is the x-percentile of the response time. For example 98% of all requests are taking less than 200ms.

### IO
To load test data services run an IOMeter or Bonnie+ load test on the servers which host the data service. For the IO subsystems look at:

	response time = queue wait time + service time

and for the linear case with constant service time

	queued = (requests per sec) - (1  / (service time in fractions of sec))

So check on a poor performing database server with “iostat -xm 5” and pay attention to the *queue size*.

### Queues
At some point shortly after the launch to production, players are complaining about very poor response times. The cpu utilization on the servers is really low. Concluding that the server and the services hosted on the server cannot cause the slowness is not correct. A blocking single-threaded server can be the cause. The requests wait in the queue and the service spends most of its time waiting for database results.

### mysql
Load test mysql servers to make sure that the configuration could handle the load with the latency distribution required. And of course out of curiosity what the peak performance was. The major goal of those load tests was to verify that the server configuration was ok. And to get a rough idea about the performance metrics.

Even simple load tests like the once described above have given very different results depending on the number of concurrent requests and the read-write ratio. Especially the read-write ratio has a big impact on the performance:
Read performance is mostly a question of the cache-hit-rate. Transactional writes (changes are persisted on commit) are way slower than in-memory cached reads.

To optimize database servers one approach is to look at the Top10 queries as number of times executed and as a total of execution time. In mysql you can use the slow query log and set the long_query_time to a very low value (as a result mysql logs all queries). Then use one of the tools to analyze the slow query log. Those tools allow to sort by different metrics. 

It is all about finding out which are the most common queries and which ones take the most resources. For the most common once either reduce the number of calls (cache result at the caller for example) or tune the query execution to the absolute minimum. In case a long running query cannot be fixed make sure it does not block a performance-critical service (for example the web client would freeze if the service does not respond in time).

## What to do if there is a server issue?
In case of a server issue; so we confirmed that the application is running smoothly and it is just this one server that has an issue, the goal is to remove or replace that particular server as soon as necessary.

First step is to find out (or just know) if the server is part of a cluster (redundant architecture) or not. Also the kind of services hosted by the server is important. A server hosting state-less web services is easier to fix than a server hosting persistent data storage.

In a cluster the participating nodes are registered on one or multiple nodes. In the best case the cluster manager marked the node as failed already. But it is very possible that the node is still considered to be good. 
Web servers are usually behind a load balancer. Quick look at the load balancer status will tell if the node was removed from the list of active nodes. If the node is still listed I remove it manually. Now get another clone/image of the server up and running and then add it to the load balancer.

For database clusters a cluster manager monitors the state of each data node. Again the node should already been removed from the cluster by the manager. A quick check and a manual fix if necessary. Adding a replacement to the cluster is more challenging than in the state-less web server case. After the cluster manager (the cluster  manager can be distributed) detects the new node a “re-balance” of the data in the cluster is initiated. Assuming the size of the data on each node is small this operation should not take very long.

For a database master in a master-readslave configuration and the master is not part of an HA-cluster make one of the read-slaves a new master, change the configuration on the appriopriate nodes to connect to this new master and change the remaining read-slaves to pull from the new master.

If the "broken server" is not part of any cluster and thus the services are only hosted on this machine then the only options we have are either fix the broken server or clone the server and replace the “broken server” with the clone. So you put the application in maintenance mode. This stops all the incoming traffic to the server. Then get to work and fix the server. When this is done bring the application out of maintenance mode and wait if things are back to normal.
