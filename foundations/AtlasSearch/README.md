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

![](/images/AtlasSearch/querylifecycle.png)

-----------------------------------------------------------------------------------------------------------------------------------------

## Search Foundations / Features

| # | Label                                                       | Description |
|:--|:------------------------------------------------------------|:-----------|
| 1 | [Basic Text Search](foundations/)     | How does text search work today using text indexes and $text/ $regex in MongoDB? |
| 2 | [Fuzzy Text Search](foundations/)     | How can we make our text indexes smarter and more efficient?|
| 3 | [Highlighting](foundations/)          | How does Atlas Search provide full text search capabilities? |
| 4 | [Autocomplete](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 5 | [Keyword](foundations/)               | What is Vector Search and how does it improve on Full Text Search? |
| 6 | [Phrase](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 7 | [Diacritics](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 8 | [Compound](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 9 | [Explain](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 10| [Custom Scoring](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 11| [Faceting](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 12| [Synonyms](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 13| [Custom Scoring](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 14| [Sorting and Pagination](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 15| [Cross-Collection Search](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 16| [Custom Analyzers](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 17| [Flexible Querying / Index Intersection](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
