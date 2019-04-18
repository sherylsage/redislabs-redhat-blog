# Building Modern Distributed Applications with Redis Enterprise on Red Hat OpenShift
by Sheryl Sage
#
This piece was written by Sheryl Sage, Director of Partner Marketing at Redis Labs.

We live in a connected world and expect that our services are always-on and instantly delivered. The Red Hat OpenShift Container Platform helps you easily build and deploy applications in nearly any public, private or multi-cloud environment. But what about building geographically distributed applications? Operating geo-distributed applications that are both accurate and efficiently distributed is extremely challenging. 

Fortunately, Redis Labs offers active-active with Conflict-Free Replicated Data Types (CRDTs) for distributed applications using [Redis Enterprise on OpenShift](https://redislabs.com/partner/redhat/). To provide some clarity on the topic, this post will cover why you need an active-active architecture, an overview of common consistency models, and how you can start building a Redis active-active environment on OpenShift.


## What is a multi-region active-active architecture?

An active-active architecture is a multi-master configuration where every database replica is a master, and can handle write operations. With an active-active database, each location has its own version, but eventually, all of them will synchronize in real time. Generally speaking you want to have a active-active architecture to improve latency for end-users, handle regional outages, and utilize platform resources better with parallel application processing. Furthermore, this architecture addresses the business requirements to store data in the region where the users reside.

In contrast, in an active-passive architecture, there is only one master with read-replicas. With active-passive, a read request will use the replica in region and a write operation is executed on the master.  

## What type of applications benefit from an active-active architecture?

Modern applications such as e-commerce, mobile, gaming, IoT, fraud mitigation and social personalization need end-to-end response times of less than 100-milliseconds, while operating databases at sub-millisecond latency at scale. This is where an active-active architecture is extremely valuable. For example, a gaming or voting application uses a distributed leaderboard.  With active-active, the votes from each region are stored locally, eventually the replicas are synchronized, and you have the same count everywhere. Another common use case is multi-region data ingest.  When you are ingesting data in different regions, you can collect the data with multiple masters at each location and eventually all the data is merged. This architecture also works well for a distributed cache.   

## Requirements for a multi-region active-active architecture

There are a couple of challenges when you talk about an active-active database - latency and conflict resolution. There is always latency because of the network, which cannot be avoided. But in the case of conflict resolution, there are three different approaches:    

* Strong consistency requires all database instances to receive messages in the same order. All databases must be updated all the time. In a distributed environment strong consistency degrades the availability and latency of application write operations. 

* Eventual consistency means that in the absence of updates, all data store replicas will converge towards identical copies of each other.  With eventual consistency, changes made to one copy of the dataset will eventually converge to be logically equivalent and propagated to all replicas, allowing all access to the data to return the last updated value. 

* Strong eventual consistency strikes a compromise between strong and eventual consistency. It guarantees that whenever two replicas have received the same set of updates -- possibly in a different order -- their view of the shared state is identical, and any conflicting updates are merged automatically to a consistent state. This allows for a decentralized deployment architecture, allowing local execution at each database instance to proceed without waiting for communication with other instances. Updates across instances will happen asynchronously with no data loss. These guarantees are tailor-made for a distributed asynchronous world.  

## How Redis Enterprise implemented Active-Active with CRDTs? 
Redis Enterprise has implemented strong eventual consistency in an [active-active architecture](https://redislabs.com/docs/active-active-datasheet/) with CRDTs. Redis Enterprise CRDTs are implemented using a global database that spans multiple clusters.  There are several benefits to this architecture:

* Seamless conflict resolution for simple as well as complex data types.

* Local latency on read and write operations, regardless of the number of geo-replicated regions and their distance from each other.  

* Even if the majority of the geo-replicated regions are down, the multi-master cluster will continue to handle read and write operations.

The most common CRDT compatible use cases are counters, activity trackers, session stores, distributed caching, inventory management. To understand how CRDTs work at a deeper level, you may enjoy reading more about [Redis Enterprise CRDTs](https://redislabs.com/docs/under-the-hood/) and their conflict resolution semantics.  

## How to use Redis Enterprise Active-Active on OpenShift
Redis Labs has partnered with Red Hat to integrate Redis Enterprise Active-Active on OpenShift, using a [Redis Enterprise Operator](https://blog.openshift.com/using-the-redis-enterprise-operator-on-openshift/) and OpenShift Route. The enhanced Redis Enterprise Operator uses the routes mechanism to expose two inter-cluster services: the Redis Enterprise Cluster API service and the DB service. Both services are used during the creation and management of an active-active CRDT deployment. 

Before you create an active-active deployment with Red Hat OpenShift Service Broker, you must create a cluster using the Redis Enterprise Cluster custom resource, with a Service Broker deployment. To configure the environment refer to the Redis Labs [documentation](https://docs.redislabs.com/latest/rs/getting-started/getting-started-openshift-crdb/).

## Conclusion
In this post you have learned why you need an active-active architecture, common consistency models, capabilities offered by Redis Enterprise, and how you can get started building a Redis Active-Active environment on OpenShift. Red Hat OpenShift provides excellent capabilities to build and deploy your distributed applications. But you still need to make sure your modern applications run seamlessly across regions with an instant response. To address these challenges Redis Enterprise Active-Active with CRDTs will help you run your distributed applications as a stateful service on OpenShift.

## Learn more
If you are interested in learning more, watch this recent [OpenShift Commons Briefing](https://youtu.be/lrOgs6xkK_U), visit the OperatorHub.io today and take the [Redis Enterprise Operator](https://operatorhub.io/operator/alpha/redis-enterprise-operator.v0.0.1) for a test run! 


