# Flexible Querying and Index Intersection with MongoDB Atlas Search

Let’s say we have an application that currently supports the querying against *items* or *quantity* and it is supported by two index defintions that look like this:


```json
{ "qty": 1 }
{ "item": 1 }
```

Now these indexes are completely separate, meaning that they will either support queries against *quantity* or queries against *item*, but not both simultaneously. For example:

```shell
db.orders.find( { "item": "abc123", "qty": { $gt: 15 } } )
```

MongoDB will leverage the *item* index and traverse its B-Tree until it eventually finds the nodes matching `abc123`. From there, ***GENERALLY*** the query optimizer will scan each node to find where the *quantity* is greater than 15. 

Now, an alternative could be to just create a compound index 
```json
{ "item":1, "qty":1 }
```

But this comes with the implications of having additional indexes and knowing ahead of time that there may be a need to add this access pattern. 
Wouldn’t it be nice to leverage the both indexes automatically if we could while writing ad-hoc queries?

------------------------------------------------------------------------------------------------------------------------------------------------------------

There exists a concept where a query engine can select the intersection of two or more indexes to more quickly and efficiently return a result set. This is called [***index intersection***](https://www.mongodb.com/docs/manual/core/index-intersection/). In practice, the query optimizer rarely selects plans that use index intersection. This is because, as of today, WiredTiger does not rank index intersection plans very high.

This is “bypassable” through MongoDB Atlas Search. Because Lucene uses inverted indexes rather than B-Tree indexes, it’s purpose-built to run queries that overlap into multiple indexes.

Let’s create an index that maps to four different fields within our `sample_mflix.movies` namespace. We’ll create mappings for `title`, `year`, `cast` and `imdb.rating`.

```json
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "cast": {
        "type": "string"
      },
      "imdb": {
        "fields": {
          "rating": {
            "type": "number"
          }
        },
        "type": "document"
      },
      "title": {
        "type": "string"
      },
      "year": {
        "representation": "int64",
        "type": "number"
      }
    }
  }
}
``` 

With this index definition, we can create an aggregation that searches for the compound of the `title` and `year` as shown below.

```json
{
  "$search": {
    "index": "<index name>",
    "compound": {
      "must": [{
          "text": {
            "query": "Fight Club",
            "path": "title"
          }
        },
        {
          "range": {
            "path": "year",
            "gte": 1999,
            "lte": 1999
          }
        }
      ]
    }
  }
}
```

Let's add a second stage after the `$search` that projects only the title and the score.
```json
{
  "$project": {
    "title":1,
    "score": { "$meta": "searchScore" }
  }
}
```
***RESULTS***

<img src="/images/AtlasSearch/17-flexible-querying-index-intersection/results.png" style="height: 75%; width:75%;"/>


What's even better about this access pattern is that if our queries were to evolve to also incorporate cast memebers and ratings, our index is still capable of
satisfying those queries. 

------------------------------------------------------------------------------------------------------------------------------------------------------------

### ***QUESTION***: How do you think this result set changes if we analyzed with the `keyword` analyzer rather than the `standard`?

Revisit the index and modify the definition to analyze the `title` field with `lucene.keyword` as shown below. When running the same aggregation your result set should change to this:

```json
{
  "analyzer": "lucene.keyword",
  "searchAnalyzer": "lucene.keyword",
  "type": "string"
}
```

<img src="/images/AtlasSearch/17-flexible-querying-index-intersection/result.png" style="height: 50%; width:50%;"/>


Credit goes to Ethan Steininger for the blog he wrote around [Flexible Querying with Atlas Search](https://www.mongodb.com/developer/products/atlas/flexible-querying-with-atlas-search/)
