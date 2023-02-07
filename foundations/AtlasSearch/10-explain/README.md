# Explain Plans in MongoDB Atlas Search

Not all search queries are created equal. Literally, there are different types of Lucene queries. If you recall from the Lucene section, 
Lucene builds out a queryTree; a tree of different query objects that represent a more complex query structure. 

<img src="/images/Lucene/queryTree.png" style="height: 70%; width:70%;"/>

In the diagram above you can see the different types of [Lucene Query Objects](https://lucene.apache.org/core/7_6_0/core/org/apache/lucene/search/Query.html)
- BooleanQuery: A Query that matches documents matching boolean combinations of other queries, e.g. TermQuerys, PhraseQuerys or other BooleanQuerys.
- WildcardQuery: Implements the wildcard search query. Supported wildcards are *, which matches any character sequence (including the empty one), and ?, which matches any single character. '\' is the escape character.
- ConstantScoreQuery: A query that wraps another query and simply returns a constant score equal to 1 for every document that matches the query. It therefore simply strips of all scores and always returns 1.
- FunctionScoreQuery: A query that wraps another query, and uses a DoubleValuesSource to replace or modify the wrapped query's score If the DoubleValuesSource doesn't return a value for a particular document, then that document will be given a score of 0.
- LatLonPointDistanceQuery: Finding all documents within a range at search time is efficient. Multiple values for the same field in one document is allowed.
- LatLonShapeQuery:
- LongDistanceFeatureQuery: 
- MultiTermQueryConstantScoreWrapper: 
- PhraseQuery: A Query that matches documents containing a particular sequence of terms. A PhraseQuery is built by QueryParser for input like "new york".
- PointRangeQuery: Abstract class for range queries against single or multidimensional points such as IntPoint.
- TermQuery: A Query that matches documents containing a term. This may be combined with other terms with a BooleanQuery.
- Default: Lucene queries that are not explicitly defined by another Lucene query are serialized using the default query

## [Explain Plans Verbosity Modes](https://www.mongodb.com/docs/atlas/atlas-search/explain/#std-label-explain-verbosity)
1. queryPlanner: Information about the query plan, excluding execution statistics
2. executionStats: Information about the query plan, including execution statistics
3. allPlansExecution: *obsolete for $search aggregations since we only have a single plan*


The plan will include
path: path to the clause
type: name of the lucene query that the operator created
analyzer: the search analyzer used with the query if specified
args: the arguments passed into the operator
stats: explain timing breakdown

If we ever need to understand ***HOW*** our query is being execute or where specifically within the query it might be bottlenecked, 
we can use an explain plan to gain insight. 


## Exercise

Using the same `$search` aggreagtion we defined in the [*9-compound*](/foundations/AtlasSearch/9-compound) exercise we can run an explain plan to see how it was executed.

```
  db.movies.explain("executionStats").aggregate([{ 
    "$search": { 
      "index": "default", "compound": { 
        "must": [{ 
          "text": { 
            "query": "wolf", "path": "title" 
          } 
        }], 
        "should": [{ 
          "text": { 
            "query": "wolf", "path": "plot" 
          } 
        }], 
        "filter": [{ 
          "range": { 
            "path": "released", "gte": ISODate("2000-01-01T00:00:00.000Z") 
          } 
        }] 
      } 
    } 
  }, 
  { 
    "$project": { 
      "title": 1, "plot": 1, "score": { 
        "$meta": "searchScore" 
      } 
    } 
  }
])    
```


I won't take the full screenshot of the result you get from running this on the `sample_mflix.movies` namespace but you will see that the explain plan includes these 3 args objects

1. This object illustrates that we are running a `TermQuery` on the `compound.must` operator, searching for the value of `wolf` over the `title` field
<img src="/images/AtlasSearch/10-explain/r1.png" style="height: 50%; width:50%;"/>

2. This object illustrates that we are running a `TermQuery` on the `compound.should` operator, searching for the value of `wolf` over the `plot` field
<img src="/images/AtlasSearch/10-explain/r2.png" style="height: 50%; width:50%;"/>

3. This object illustrates that we are running `PointRangeQuery`s on the `compound.filter` operator, searching for dates greater than or equal to Jan 1, 2000
<img src="/images/AtlasSearch/10-explain/r3.png" style="height: 50%; width:50%;"/>


***NOTE*** You can additionally see the nanoseconds taken to perform the query for each object as well as for the whole aggregation 
> `nanosElapsed`: nanoseconds elapsed for the particular search object

> `executionTimeMillisEstimate`: estimated milliseconds elapsed for the entire aggregation
