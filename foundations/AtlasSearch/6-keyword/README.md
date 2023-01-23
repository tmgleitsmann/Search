# Searching for Keywords in MongoDB Atlas Search
Up to this point in the repository, we've been talking about breaking up sequences of texts into arrays of "tokens". This is perfect if we want to query against a series of tokens with a set of tokens as input. But what if we want to match for exact terms?

This could be:
- usernames
- emails
- zip codes
- full names

Anything where instead of multiple tokens, we'll want to query for an individual token, without alteration.

> Example: `The quick brown fox jumped over the lazy dog` would tokenize to `[The quick brown fox jumped over the lazy dog]` rather than traditionally `[quick, brown, fox, jump, over, laz, dog]`

**NOTE: The Keyword Analyzer does support case-insensitive search as it does not apply a lowercase token filter.**

## Exercise
Continuing to work off of the cluster we've already built with the dataset we've already loaded in the previous exercises, we're going to create a new index for keyword.

### Creating the Keyword Index

1. Create a new index on the `sample_mflix.movies` namespace titled `keyword`. 

      <img src="/images/AtlasSearch/5-autocomplete/namespace.png" style="height: 40%; width:40%;"/>
      
2. Refine the index. We will want to get a bit more granular than accepting the default `dynamic` index settings provided by Atlas out of the box. Plus, we'll want to utilize the keyword lucene analyzer rather than the default.

      <img src="/images/AtlasSearch/5-autocomplete/field-mappings.png" style="height: 70%; width:70%;"/>
      
3. Modify the field mapping definition to be of type `string` against the `cast` field. Since this field is a string, we can turn off the dynamic toggle.
  - Change the index and search analyzer to `keyword`
  - Index Options: `docs` --> specifies the amount of information to store for the indexed field.
    - `docs` --> only indexed documents
    - `freqs` --> only indexed documents and term frequency
    - `positions` --> only indexed documents, term frequency and term positions
    - `offsets` --> index documents, term frequency, term positions and term offset
  - Store: `false` --> lucene allows you to store the exact document's text as well as what's analyzed. Since the `keyword analyzer` doesn't change the           text, this would be redundant to have set as true. 
  - Norms: `false` --> String that specifies whether to include or omit the field length in the result when scoring. The field length is used as a               tiebreaker in the case that documents have matching scores. The shorter field length will return first.
  
      <img src="/images/AtlasSearch/5-autocomplete/field-mappings.png" style="height: 70%; width:70%;"/>
  
### Creating the Keyword Aggregation

1. Navigate to the sample_mflix.movies namespace in the collections tab and then click on `aggregation`

    The first stage will be for `$search`. We'll define the `keyword` index we intend on using.
    We're going to search for the movies starring `Chris Pratt`. I'm expecting to match against the token `Chris Pratt` which exists within some             document's `cast` field. 
  
    ```json
      {
        "$search":{
          "text": "keyword",
          "autocomplete": {
            "query": "Chris Pratt",
            "path": "cast"
          }
       }
      }
     ```
      
      <img src="/images/AtlasSearch/5-autocomplete/searchStage.png" style="height: 40%; width:40%;"/>
      
2. The next stage we'll use is `$project` to cleanup our payload. 

      
    ```json
      {
        "$project":{
          "_id":0,
          "title":1,
          "year":1,
          "score": { "$meta": "searchScore" }
        }
      }
    ```
    
    
    <img src="/images/AtlasSearch/5-autocomplete/projectStage.png" style="height: 40%; width:40%;"/>
  
    Here you'll see we are matching for movies with Chris Pratt. 
    
    <img src="/images/AtlasSearch/5-autocomplete/results.png" style="height: 60%; width:60%;"/>
    
-------------------------------------------------------------------------------------------------------------------------------------------------------
  
***NOTE***: Remember that the keyword analyzer is case sensitive. To test this, try searching for `chris pratt` and see if you match for any results.

```json
  {
    "$search":{
      "text": "keyword",
      "autocomplete": {
        "query": "chris pratt",
        "path": "cast"
      }
    }
  }
```
