# Elasticsearch Architecture

Elasticsearch is an open-source, near real-time distributed storage, search and analysis engine. 
It runs very similarly to MongoDB in that you have a cluster composed of a series of nodes that are ultimately responsible for searching against 
JSON documents. However, there’s more to it than just distributing your data for full text search. Let’s get into what constitutes Elasticsearch. 

1. Application Search: There is an application search component that elasticsearch supports when it comes to bringing data closer to an end user. What does that mean? This means, allowing users to search via a more ambiguous query than what you would traditionally see against a database to find a subset of relevant documents. You can think of application search like Google, Amazon, or any catalog use case where an end user can interact with a search bar, filters and facets. 
2. Analytics Platform: Elasticsearch supports aggregation over structured data to generate charts and reports. You generally see this with logging or system metrics use cases and it’s more generally referred to as Application Performance Management. It is a very common use case that leverages the Elastic Stack (will not be covered).
3. Machine Learning: The Elastic Stack supports forecasting the future or understanding anomalies.


We are going to focus ***PURELY*** on application search because these are the workloads we are looking to target with Application Search. 
MongoDB Atlas Search is a poor fit for analytics and machine learning use cases today. 

You may hear the term *Elastic Stack*. This is a reference to all the technologies offered by elastic, and they synergize very well together. 
Outside of elasticsearch, we have:
- X-Pack: Pack of features that add functionality to elasticsearch and kibana 
- Beats: Lightweight agents that ship data to logstash or elasticsearch
- Logstash: Data processing pipeline that processes logs from applications before being shipped downstream
- Kibana: An analytics and visualization platform that allows you to visualize data from elasticsearch

To keep this repository simple, we will not be covering these technologies listed above, as they are not required for application search. 

------------------------------------------------------------------------------------------------------------------------------------------------------------

Data is stored as JSON documents in Elasticsearch, just like in MongoDB. 
It scales very well which allows Elasticsearch to scale and stay lightning fast at query time. 

<img src="/images/Elasticsearch/elasticsearch_architecture.png" style="height: 35%; width:35%;"/>

In this elasticsearch diagram you can see we have components like
- Cluster(s)
- Nodes
- Index and Shards
- Documents
Let’s dive into what each of these are. 

------------------------------------------------------------------------------------------------------------------------------------------------------------

### Cluster
A cluster in Elasticsearch, just like in MongoDB, is just a set of nodes that store your documents and indexes. 

### Nodes
A node is a single server in the cluster. It stores *shard(s)*, which are just distributed *fragments* of index(es). 
Unlike MongoDB where data is replicated at the node level and nodes are assigned their roles as primary, secondary, read-only, etc. 
this is done at the *shard level* in elasticsearch. 

### Index
An index is just a collection of documents. 

### Shard
A shard is essentially a piece of an index. Shards allow us to take pieces of an index and horizontally distribute them across a cluster. 
It is the mechanism that allows for efficient parallel processing and replication. We assign shards to be primary vs replicas rather than the nodes themselves.
We'll cover more about shards soon.

### Documents
As state previously, documents are just the JSON entities that encompass the data that we are indexing. 

------------------------------------------------------------------------------------------------------------------------------------------------------------

By sharding the Elasticsearch index, we are capable of balancing each Lucene segment, similar to how MongoDB balances chunk sizes in sharded clusters. 
With Elasticsearch’s sharded architecture you can scale shards for search throughput, remain resilient, and theoretically infinitely scale storage. 

<img src="/images/Elasticsearch/shard_diagram.png" style="height: 45%; width:45%;"/>

------------------------------------------------------------------------------------------------------------------------------------------------------------

With the diagram below you can see how multiple indexes, existing on multiple shards can be configured against a 3-node cluster. 

1. The top cluster state is Green, meaning that all shards are in a healthy state.

2. The middle cluster state is Yellow, meaning that not all shards are in a healthy state, but it is still functional.

3. The last cluster state is Red, meaning one or more primary shards are inoperable. In this case, it’s shard 2.

<img src="/images/Elasticsearch/cluster_states.png" style="height: 45%; width:45%;"/>

At this point we’ve established that there are two kinds of shards; Primary and Replica. The purpose of replicating shards is purely for resiliency.  
You can distribute your index into multiple shards which is what we’ve done here with shards 1, 2 and 3.

Similarly to MongoDB, write operations happen first on the primary shard before being replicated to the replicas. Read operations can occur on either. 

------------------------------------------------------------------------------------------------------------------------------------------------------------

## Node Types

There are still node types. This is not an [exhaustive list](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html). 
- Master Node: in charge of lightweight cluster-wide actions such as creating or deleting an index, tracking which nodes are part of the cluster, and deciding which shards to allocate to which nodes. 
- Data Node: hold the shards that contain the documents you have indexed. Data nodes handle data related operations like CRUD, search, and aggregations.
- Client Node/Coordinating Node: route requests, handle the search reduce phase, and distribute bulk indexing. Essentially, coordinating only nodes behave as smart load balancers.






article references:
[different-elasticsearch-components-and-what-they-mean-in-5-mins](https://devopsideas.com/different-elasticsearch-components-and-what-they-mean-in-5-mins/)
[what-is-apache-lucene-that-powers-elasticsearch](https://lakshyabansal.hashnode.dev/what-is-apache-lucene-that-powers-elasticsearch)
