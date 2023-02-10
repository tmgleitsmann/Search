# Custom Scoring in MongoDB Atlas Search

The default [scoring mechanism](https://lucene.apache.org/core/3_5_0/scoring.html) with Lucene doesn't always cut it. Often times with more comprehensive searches, the developer needs to define more comprehensive scoring. 
Take the example in the [*compound lesson*](/foundations/AtlasSearch/9-compound). We are using a combination of different clauses to evaluate documents that Lucene deems to be relevant. 
But should each of these clauses be treated equally? If a movie title has the token I'm searching for, it COULD be a great indicator that the movie is about that token.
If a movie plot has a token I'm searching for, it could also be a great indicator that the movie is about a token, but less so than matching on the title field. 

We may want to take additional steps in order to further influence our document scores through 
- The position of the search term in the document
- The frequency of occurrence of the search term in the document,
- The [operator(s)](https://www.mongodb.com/docs/atlas/atlas-search/operators-and-collectors/#std-label-fts-operators) and/or [analyzer](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/#std-label-analyzers-ref) the query is using

With MongoDB Atlas Search, developers have a handful of [score modifying options](https://www.mongodb.com/docs/atlas/atlas-search/scoring/#std-label-scoring-function) available to all operators. 
1. Boost: multiplies a result's base score by a given number or the value of a numeric field in the documents
    > Will take a `value` or a `path` to boost by
2. Constant: replaces the base score with a specified number
    > Just takes a `value`
3. Embedded: Aggregate scores from multiple matching embedded documents 
4. Function: specify the numeric field for computing the final score through an expression



## Boost Example 

Using the index and aggregation we created in [lesson 9](/foundations/AtlasSearch/9-compound), we're going to double the score from matching against `title`.
  > ` "score": { "boost": { "value": 2 } } `
  is going to slot in the `must` operator. It will double the score from matching against `title` field. 
 
 ```json
    {
        "compound": {
          "must":[{
            "text":{
              "query":"wolf",
              "path":"title",
              "score": { "boost": { "value": 2 } }
            }
          }]
        }
    }
 ```
 
 You should immediately see a difference in the scores. For example, `Wolf Summer` now returns a score of `10.3457` when in lesson 9 it returned a score of `6.9133`
 
 
 ## Constant Example 
 
 Similarly, using the same index and aggreagtion we're going to reduce the significance of matching tokens against the `plot` field. When we come across against a match, we are going to influence the score by modifying it to be a constant value of `1`.
 
 > ` "score": { "constant": { "value": 1 } } `
  is going to slot in the `should` operator. It will double the score from matching against `plot` field. 
 
 ```json
    {
        "compound": {
          "should":[{
            "text":{
              "query":"wolf",
              "path":"plot",
              "score": { "constant": { "value": 1 } }
            }
          }]
        }
    }
 ```
 
 You should see that the documents in the result set with the highest scores are ones with Wolf and ONLY Wolf in the `title`. It is irrespective of `plot` because we effectively buried the score associated with the `plot` field. 
 
  ## Embedded Example  - TODO (no embedded fields in the sample_mflix dataset)
  
  ## Function Example 
  
  You have a plethora of different options when it comes to building out function scores. 

  - Arithmetic Expressions: Add or multiply an array of expressions
  - Constant Expressions: Allow a constant number in the function score
  - Gaussian Decay Expressions: Reduces the score by multiplying at a specified rate. The Gauss function is a bell shape that decays slowly, then rapidly,      then slowly. 

       <img src="/images/AtlasSearch/11-scoring/gauss-decay-expression.png" style="height: 50%; width:50%;"/>
       <img src="/images/AtlasSearch/11-scoring/gauss-diagram.png" style="height: 50%; width:50%;"/>
  - Path Expressions: Utilize the value within a path
  - Score Expressions: Represents the relevance score, is the same as the current score of the document (default scoring)
  - Unary Expressions: Expression taking a single argument. Today you can use this to find the `lo10(x)` or `log10(x+1)` of a specified x.
        <img src="/images/AtlasSearch/11-scoring/log10.png" style="height: 50%; width:50%;"/> 

    
    ### Arithmetic
    Let's adjust our scoring against the `title` field to multiply against the imdb rating of the movie. This way, not only will we favor movies with matching tokens in the `title`, but we'll also be favoring GOOD movies. 
    
    ```json
    {
        "compound": {
          "must":[{
            "text":{
              "query":"wolf",
              "path":"title",
              "score": { 
                "function": {
                  "multiply": [{
                    "path": {
                      "value": "imdb.rating",
                      "undefined": 2
                    }
                  },
                  {
                    "score": "relevance"
                  }] 
                } 
              }
            }
          }]
        }
     }
     ```

    ### Constant
    
    We got a taste for this sort of operation above in the ***Constant Example*** but if we wanted to accomplish this within function scoring it would look like this. 

    ```json
    {
        "compound": {
          "should":[{
            "text":{
              "query":"wolf",
              "path":"plot",
              "score": { "function": { "constant": 1 } }
            }
          }]
        }
    }
    ```


    ### Function











ref: https://www.elastic.co/guide/en/elasticsearch/guide/current/decay-functions.html
