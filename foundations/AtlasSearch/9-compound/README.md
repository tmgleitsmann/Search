Recall that Lucene will create a query tree, comprised of potentially many different query objects (ie. term query, wildcard query, fuzzy query, etc.) before evaluating the query and scoring. What the `compound` operator allows us to do is create these more complex trees and then combine the operations of each leaf node while traversing up the tree. 

<img src="/images/Lucene/queryTree.png" style="height: 70%; width:70%;"/>

Each element of a compound query is called a clause, and each clause consists of one or more sub-queries.

The score for the result set is calculated by summing the score that every document received for each clause and is then sorted highest to lowest. 

The individual clauses in a `compound` statement can have different behaviors
- must: Clauses that must match to for a document to be included in the results.
- mustNot: Clauses that must not match for a document to be included in the results. This operator won’t contribute to the document score. 
- should: Clause that you’d prefer to match with your result set, but don’t necessarily have to. Documents that match the should clause should also yield higher scores. 
- filter:  Clauses that must all match for a document to be included in the results. Very similar to the must clause, however filter does not contribute to the score. 


## Exercise
In this exercise we are going to run searches across the `title`, `released` and `plot` fields. We will search for words across both `title` and `plot` and then filter based off of a range of dates.
We'll search for movies released after the year 2000 that MUST contain the token `wolf` in the title and SHOULD contain it as well in the plot.

### Creating the Index
Follow steps 1-3 under [*Configure Atlas Search Index*](/foundations/AtlasSearch/2-basic) if you don't already have the dynamic, default full text search index set on the `sample_mflix.movies` namespace.

### Creating the Wildcard Aggregation

1. Navigate to the sample_mflix.movies namespace in the collections tab and then click on `aggregation`

    The first stage will be for `$search`. We'll define the `default` index we intend on using.
    We're first going to swap out the `text` operator for the `compound` operator to construct our query clauses. Then we are going to create the `must` clause, querying
    against `wolf` on the `title` field. Next we are going to creat the `should` clause, querying against `wolf` on the `plot` field. Lastly, we're going to create the filter
    clause, using the [range](https://www.mongodb.com/docs/atlas/atlas-search/range/) oeprator to query for dates after Jan 1, 2000 against the `released` field.
    
    ```json
    {
      "$search": {
        "index": "default",
        "compound": {
          "must":[{
            "text":{
              "query":"wolf",
              "path":"title"
            }
          }],
          "should":[{
            "text":{
              "query":"wolf",
              "path":"plot"
            }
          }],
          "filter":[{
            "range": {
              "path": "released",
              "gte": ISODate("2000-01-01T00:00:00.000Z")
            }
          }]
        }
      }
    }
    ```
    <img src="/images/AtlasSearch/9-compound/searchStage.png" style="height: 50%; width:50%;"/>
    
2. The next stage we'll use is `$project` to cleanup our payload. 

      
    ```json
      {
        "$project":{
          "title":1,
          "plot":1,
          "score":{"$meta":"searchScore"}
        }
      }
    ```
    
    
    <img src="/images/AtlasSearch/9-compound/projectStage.png" style="height: 50%; width:50%;"/>
    
--------------------------------------------------------------------------------------------------------------------------------------------------------

Here you'll see the results we match against. One thing to note is the score is the same for a lot of documents. The title scoring is just as influential as the plot scoring with this query and we may not want that to be the case. What we'll be exploring later is custom scoring for these different query clauses to ensure we're scoring the most relevant documents the highest. 
    
<img src="/images/AtlasSearch/9-compound/results.png" style="height: 70%; width:70%;"/>
    
