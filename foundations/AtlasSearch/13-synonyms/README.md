# Synonyms with MongoDB Atlas Search

I recall when I would spend multiple months abroad at a time that even the simplest of words would escape my vocabulary once I returned home. 
There was one time in particular after coming home to Texas from France where I forgot how to say "trash can". I knew what I wanted to say, I knew how to describe it,
but I didn't know the term anymore. And this can be problematic if I need to match against tokens in a corpus. 

Luckily, Lucene allows us to configure our indexes to map against synonyms as if it had access to a thesaurus. So instead of searching for the tokens "trash can", 
I could alternatively search for "bin" and get the same result set. 

Here is how this logic might look as a finite state machine. Books can be mapped with guides and rocks can be mapped with stones in this example. 

<img src="/images/AtlasSearch/13-synonyms/fsm.png" style="height: 50%; width:50%;"/>

------------------------------------------------------------------------------------------------------------------------------------------------------

How might this work under the hood though? Are we applying synonyms at index time or at query time? And what are the implications of doing one way over the other?

- Query Time: Query time synonyms expand terms used in a query and are therefore able to match more documents in the index.
- Index Time: Index time synonyms expand the terms used at the point of indexing. The index itself then contains more terms, making documents more easily matchable.

***Index Time*** synonyms may look to be the more appealing option based off of these descriptions, however there are some major caveats. 
- The synonym list cannot be updated without reindexing everything, which is very inefficient in practice.
- The search score would be impacted because synonym tokens are counted as well.
- The indexing process becomes more time-consuming and the indexes will get bigger. It is negligible for small data set but is very significant for big ones.

***Query Time*** synonyms move all the additional complexities to query time. This means that instead of searching for a token, you're searching for that token as well as all of its synonyms by constructed a more complex lucene query under the hood. 
- This additional complexity can add latency to the query. 

------------------------------------------------------------------------------------------------------------------------------------------------------

MongoDB Atlas Search uses the latter (Query Time). There are 3 major steps to incorporating Synonyms in your Atlas Search Queries. 
1. Configuring the Synonyms against the Index. *this still isn't index time definition*
2. Creating the Synonyms Collection in MongoDB
3. Modifying our Queries to incorporate the Synonyms

<img src="/images/AtlasSearch/13-synonyms/steps.png" style="height: 50%; width:50%;"/>

------------------------------------------------------------------------------------------------------------------------------------------------------

## Synonyms Exercise

### Creating The Synonyms Collection
The very first thing we'll want to do is create a synonyms collection. This will be a set of documents that define which tokens need to be treated as synonyms, and whether they should be `equivalent` mapped or `explicitly` mapped.
- Equivalent: describes a set of tokens that are equivalent to one another. 
- Explicitly: matches input tokens and replaces them with all alternative synonyms tokens.

1. Connect to the MongoDB Atlas Cluster
2. With the sample dataset already loaded, switch over to the `sample_mflix` database. Then create the `movies_synonyms` collection.

```shell
use sample_mflix
```

```shell
db.createCollection("movies_synonyms")
```

3. Insert our synonyms document. This will *equivalently* map tokens like book, novel, comic, manual, ledger and guide together. 

```shell
db.movies_synonyms.insertOne({ "mappingType": "equivalent", "synonyms": [ "book", "novel", "comic", "manual", "ledger", "guide"] })
```

### Creating The Index

Next we'll want to create an index that takes into account synonyms we previously created. Navigate to the ***Search*** tab in Atlas and create a new, default index on the `sample_mflix.movies` namespace. 

1. Define the `sample_mflix.movies` namespace within the visual editor.

<img src="/images/AtlasSearch/13-synonyms/index1.png" style="height: 50%; width:50%;"/>

2. Refine the index and navigate to *Synonyms Mapping* section.

<img src="/images/AtlasSearch/13-synonyms/index2.png" style="height: 50%; width:50%;"/>

3. Give your synonyms mapping the name `movies_synonyms`, and assign it to the `movies_synonyms` collection. We'll keep the analyzer the same as how the text sequences will be tokenized with the documents. 

<img src="/images/AtlasSearch/13-synonyms/index3.png" style="height: 50%; width:50%;"/>


### Creating a Search Query

Lastly let's put this to the test and construct a $search aggreagtion that leverages these synonyms. 

Navigate to the `Aggregation` tab within `Collections>sample_mflix>movies` namespace in the Atlas UI. Lets build out the `$search` aggregation. We want to look for documents that match for the query `guide`. 

<img src="/images/AtlasSearch/13-synonyms/search.png" style="height: 50%; width:50%;"/>

```json
{
 "$search": {
  "index": "default",
  "text": {
   "query": "guide",
   "path": "title",
   "synonyms": "movies_synonyms"
  }
 }
}
```

Run the aggregation and evaluate the result set. To make it easier to consume, you can add in a project stage for only the movie titles. 

```json
{
  "$project":{
    "title":1
  }
}
```

You'll notice our result set doesn't necessarily contain documents corresponding to JUST guide. We are also matching on its synonyms. 

<img src="/images/AtlasSearch/13-synonyms/results.png" style="height: 50%; width:50%;"/>

------------------------------------------------------------------------------------------------------------------------------------------------------

Additional Resources:
- [How to use the Synonyms Feature Correctly in ElasticSearch](https://towardsdatascience.com/how-to-use-the-synonyms-feature-correctly-in-elasticsearch-7bdf856a94cb)
- [Synonyms in Solr I — The good, the bad and the ugly](https://medium.com/empathyco/synonyms-in-solr-i-the-good-the-bad-and-the-ugly-efe8e437a940#:~:text=Query%20time%3A%20Query%20time%20synonyms,can%20be%20more%20easily%20matched.)
