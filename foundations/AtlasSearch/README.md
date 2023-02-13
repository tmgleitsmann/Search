# MongoDB Atlas Search

The MongoDB Atlas Developer Data Platform provides a means to perform full text searches across data that exists within a MongoDB Atlas cluster. 

Traditionally, developers would have to maintain the transactional database, the Extract, Transfer and Load (ETL) process, and the search engine itself.
This meant multiple technologies governing multiple copies of the same data and being exposed to multiple query languages. This can be confusing and complicated to maintian 
and can introduce problems around
- consistency of data : Eventual consistency in data replication from the transactional database to the search engine
- management of infrastructure : More infrastructure and data processing
- increased costs : More infrastucture + data duplication

![](/images/AtlasSearch/atlassearcharchitecture1.png)

How does MongoDB Atlas manage this process for you? In laymans terms, MongoDB starts up a Lucene process on the same Atlas Nodes that host your data, and expose it through a set of APIs. 
We package the Lucene and the APIs into a module we call "Atlas Search Module" (ASM) that gets downloaded and installed upon the creation of the first search index. This way we can build out 
the Lucene Inverted Indexes and have them point back to where the corresponding data exists on disk. 

![](/images/AtlasSearch/atlassearcharchitecture2.png)

For every new index created, the ASM will build out the inverted index by performing a full collection scan against the corresponding data. This happens as a background process,
so you are still capable of supporting transactional workloads against the cluster. However do be mindful of the resource consumption required to build out the index. This can
take quite a bit of time, memory and CPU.

![](/images/AtlasSearch/initialindexing.png)

Once the index is initial constructed, a change stream will be constructed against the namespace being indexed that captures all the inserts, updates and deletes and send them to the ASM. 

![](/images/AtlasSearch/incrementalindexing.png)


-----------------------------------------------------------------------------------------------------------------------------------------
## Query Lifecycle

When the client sends a full text search query it will immediately get routed to the mongod process. This process will then recognize that the first stage of the aggregation is a `$search` stage and pass it along to the `mongot` process (referred to as ASM above). The mongot process will convert our `$search` stage into a Lucene query and retrieve the ObjectIDs of the documents Lucene deems relevant from the inverted index before passing them back to the mongod process. The `mongod` process then has to do the ObjectID lookup operations to retrieve the docuements from disk. 

![](/images/AtlasSearch/querylifecycle.png)

-----------------------------------------------------------------------------------------------------------------------------------------

## Search Foundations / Features

| # | Label                                                       | Description |
|:--|:------------------------------------------------------------|:-----------|
| 1 | [Building a Search Engine](1-engine)          | Build a Python Search Engine in Code|
| 2 | [Basic Text Search](2-basic/)     | Run the default full text search |
| 3 | [Fuzzy Text Search](3-fuzzy/)     | Run text search that can handle `n` amounts of character mispellings|
| 4 | [Highlighting](2-basic/)          | Highlight the tokens matched during the search |
| 5 | [Autocomplete](5-autocomplete/)          | Run real time text inference of token(s) |
| 6 | [Keyword](6-keyword/)               | Run exact, case-senstive searches of tokens |
| 7 | [Phrase](7-phrase/)          | Run searches over ordered sequence of tokens |
| 8 | [Wildcard](8-wildcard/)          | Enables queries which use special characters in the search string that can match any character |
| 9 | [Compound](9-compound/)          | Run a search that combines two or more operators into a single query (or clause) |
| 10| [Explain](10-explain)       | Understand how the mongot (lucene) returns results in order to tune performance |
| 11| [Custom Scoring](11-scoring)         | Implement custom relevance weights where some fields more important than other fields |
| 12| [Faceting]        | Dynamically cluster search results into categories |
| 13| [Synonyms]       | Search not just for matching tokens, but also for statically mapped tokens (token synonyms) |
| 14| [Sorting and Pagination]       | Tips and tricks to sort or paginate over a result set quicly |
| 15| [Cross-Collection Search]         | Run text search that spans across multiple collections |
| 16| [Custom Analyzers]         | Build out a customer analyzer rather than selecting one of the default Lucene ones |
| 17| [Flexible Querying / Index Intersection]         | Combine multiple search indexes to perform performant and dynamic queries |
