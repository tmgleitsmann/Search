# Searching for Phrases in MongoDB Atlas Search
With Search so far the order of the tokens we've been searching for hasn't mattered. But what if we want to favor tokens in a particular order? The phrase operator performs search for documents containing an ordered sequence of terms.

This is great for searches against:
- Titles
- Geography (multiple tokens ie. New York)
- Phrases 


Anything where we want to match against multiple tokens in a particular order. The distance between the tokens is configurable through the `slop` field.

## Exercise
Continuing to work off of the cluster we've already built with the dataset we've already loaded in the previous exercises, we're going to create a new index for phrases.

### Creating the Phrase Index

1. Create a new index on the `sample_mflix.movies` namespace titled `phrase`. 

      <img src="/images/AtlasSearch/6-keyword/index-config1.png" style="height: 40%; width:40%;"/>
      
2. Phrase doesn't need a special index configuration, so a default dynamic index will do. For good practice though we'll refine our index against just the `title` field. 
  
    <img src="/images/AtlasSearch/6-keyword/field-mappings.png" style="height: 40%; width:40%;"/>
    
### Creating the Phrase Aggregation

1. Navigate to the sample_mflix.movies namespace in the collections tab and then click on `aggregation`

    The first stage will be for `$search`. We'll define the `phrase` index we intend on using.
    We're going to search for `Harry Potter` moves. Normally the token order wouldn't matter, but since we're going to be defining `slop` to be 0, the words must be in the exact same position as the query in order to be considered a match. Use the `phrase` operator rather than the `text` operator. 
    
     ```json
      {
        "$search":{
          "index": "phrase",
          "phrase": {
            "query": "Harry Potter",
            "path": "title",
            "slop":0
          }
       }
      }
     ```
    
    <img src="/images/AtlasSearch/6-keyword/searchStage.png" style="height: 40%; width:40%;"/>
    
2. The next stage we'll use is `$project` to cleanup our payload. 

      
    ```json
      {
        "$project":{
          "title":1,
          "score": { "$meta": "searchScore" }
        }
      }
    ```
    
    <img src="/images/AtlasSearch/6-keyword/projectStage.png" style="height: 40%; width:40%;"/>
    
    Here you'll see we are matching for Harry Potter movies. 
    
    <img src="/images/AtlasSearch/6-keyword/results.png" style="height: 60%; width:60%;"/>
    
    
    
    
