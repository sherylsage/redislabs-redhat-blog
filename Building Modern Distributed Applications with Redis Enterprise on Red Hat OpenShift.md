# Building Modern Distributed Applications with Redis Enterprise on Red Hat OpenShift
Sheryl Sage, Director Partner Marketing at Redis Labs
#
We live in a connected world and expect that our services are always-on and instantly delivered. The Red Hat OpenShift Container Platform helps you more easily build and deploy applications in nearly any public, private or multi-cloud environment. But what about the demands of geographically distributed applications? Writing distributed applications that are both correct and well distributed is extremely challenging. 

Fortunately, Redis Labs recently introduced active-active with Conflict-Free Replicated Data Types (CRDTs) for distributed applications using Redis Enterprise on OpenShift. To provide some clarity on the topic, this post will cover:

  - Why use an active-active architecture
  - Common consistency models
  - How Redis Enterprise implemented active-active with CRDTs
  - Getting started with a Redis Active-Active environment on OpenShift
  - Drag and drop images (requires your Dropbox account be linked)

## What is an active-active architecture?

When enterprises deploy applications across multiple data centers (or cloud regions), one approach is to build an active-active architecture. Active-active is a multi-master setup where every database replica is a master, and can handle write operations. This allows you to service a global set of users while processing locally, handle regional outages, and utilize platform resources better with parallel application processing.

## Why you need an active-active database?

In the digital age, you may be shopping on your phone in Munich while taking a train to Austria, and the expectation is that your application is always-on and the session store data is continuously available for the entire trip. Furthermore, modern applications such as e-commerce, mobile, gaming, social, IoT, fraud mitigation and social personalization also need response times of less than 100-milliseconds, while operating at sub-millisecond latency at scale. These modern applications frequently have multiple databases, located in-region. For example, a gaming or voting applications uses a distributed leaderboard where votes are stored locally, but eventually the replicas are synchronized and you have the same count everywhere. Another common example is multi-region data ingest applications.  When you are ingesting data in different regions, you can collect the data with multiple masters at each location. You ingest the data and eventually, all the data is merged. Finally, Redis Enterprise is frequently to distribute your cache in several data centers and with active-active the cache is eventually synchronize in real time.  

## Requirements for an active-active architecture
There are a couple problems when you talk about an active-active database - latency and conflict resolution. There is always latency because of the network, which cannot be avoided. But in the case of conflict resolution, there are three different approaches:  


Dillinger uses a number of open source projects to work properly:

* Eventual Consistency - means that in the absence of updates, all data store replicas will converge toward identical copies of each other.  With eventual consistency, changes made to one copy of the dataset will eventually converge to be logically equivalent and propagated to all replicas, allowing all access to the data to return the last updated value.  
* Strong eventual consistency - strikes a compromise between strong and eventual consistency. It guarantees that whenever two replicas have received the same set of updates -- possibly in a different order -- their view of the shared state is identical, and any conflicting updates are merged automatically to a consistent state. This allows for a decentralized deployment architecture, allowing local execution at each database instance to proceed without waiting for communication with other instances. Updates across instances will happen asynchronously with no data loss. These guarantees are tailor-made for a distributed asynchronous world.  
* Causal consistency mode - includes a notion of causality, corresponding to the order in which write operations are executed to the data set. Two write operations are causally related if the execution of one write operation possibly influences the value written by the second write operation. 

## How Redis Enterprise Implemented Active-Active with CRDT? 
Redis Enterprise has implemented strong eventual consistency and causal consistency in Active-Active deployments with CRDTs. Redis Enterprise CRDTs are implemented using a global database that spans multiple clusters. This architecture provides for local latency on read and write operations, regardless of the number of geo-replicated regions and their distance from each other, conflict resolution for simple as well as complex data types and if the majority e.g., 3 of 5 of geo-replicated regions are down, the remaining geo-replicated regions can continue to handle read and write operations.

To understand Redis CRDTs and their conflict resolution semantics there are several things to keep in mind.  The most common CRDT compatible use cases are counters, activity trackers, session stored, distributed caching, inventory management, and there are many-many more. Nevertheless, there are some applications such as financial transactions, order processing and fulfillment that are not good use cases and are not compatible. It's very important that when you develop your application, you keep this in mind.  In these cases you cannot stop using some of the traditional databases. 

Furthermore, developers should assume that conflicts may happen in the background, and your application should be stable and resilient to any conflict. You cannot cannot override the default conflict resolution semantics, so last writer wins goes with the string and there is a conflict-free merge that goes with a counter. All of this is built into the data type because that is how CRDTs work. For example, if you have three data centers Atlanta, Barcelona, and Chicago each with three different replicas of Redis Enterprise, each with their own cluster replica. Each cluster is a master database, so you can write into each.  When you increment a voting app counter by 10 in Atlanta and increment by 20 in Barcelona and 30 in Chicago, every node takes an update. Eventually, the counter should have a value of 60.  So you need to know this conflict resolution semantic. The conflict here is that we updated the same counter simultaneously in three different locations and they had to synchronize. In this case CRDT tracks the counter data type and the conflict is resolved.

## How to use Redis Enterprise Active-Active on OpenShift
Redis Labs has partnered with Red Hat to integrate Redis Enterprise Active-Active on OpenShift The enhanced Redis Enterprise Operator uses the routes mechanism to expose two inter-cluster services: the Redis Enterprise Cluster API service and the DB service. Both services are used during the creation and management of an active-active CRDT deployment. 

Before you create an active-active deployment with Red Hat OpenShift Service Broker, you must create a cluster using the Redis Enterprise Cluster (REC) custom resource, with a Service Broker deployment. Get started [here](https://docs.redislabs.com/latest/rs/getting-started/getting-started-openshift-crdb/).

## Conclusion
In this post you have learned why you need an active-active architecture, common consistency models, capabilities offered by Redis Enterprise, and how you can get started building a Redis Active-Active environment on OpenShift. Red Hat OpenShift provides excellent capabilities to build and deploy your distributed applications. But you still need to make sure your modern applications run seamlessly across regions with blazing fast speed.  

Redis Enterprise runs seamlessly as a stateful service on OpenShift providing an efficient and highly scalable and resilient database for cloud-native applications.

## Learn more
[Register](https://redislabs.com/webinars/easily-build-deploy-real-time-applications-redis-enterprise-red-hat-openshift-container-platform/) for a joint webinar “Easily Build and Deploy Real-time Applications with Redis Enterprise on Red Hat OpenShift Container Platform”, on March 29th with Red Hat’s Rob Szumski and Adi Foulger, Principal Architect for Redis Labs.  
