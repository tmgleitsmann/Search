# Building Autocomplete within Atlas Search

Autocomplete or "*type-ahead*" in full text search is the ability to infer words or phrases based off the input of an incomplete string. It leverages a different, more *expensive* index that will tokenize text sequences based off of the following tokenizers
- edgeGram: Generate groups of tokens without moving the cursor from the first, leftmost edge
- rightEdgeGram: Edge Gram, but the cursor begins from the right side of a text sequence rather than left
- nGram: Generates groups of tokens while moving the cursor from the first, leftmost edge

For example, given the string `men with knives`
min token size: 3
max token size: 5

| tokenizer    | T1 | T2 | T3 | T4 | T5 | T6 | T7 |
|:-------------|:---|:---|:---|:---|:---|:---|:---|
|   edgeGram   | men |wit | with |  kni|kniv | knive| X|
|   nGram      | men |wit | with | kni|kniv | knive|  nives |
|rightEdgeGram | men |ith |with|ves  |ives | nives| X |

So as someone types, these tokens will be matched against in order to retrieve relevenat documents by keystroke. 

You could imagine that for traditional autocomplete use-cases where someone is typing from left to right 
> ie. searching for the name of a product

that edgeGram would be the best tokenizer, as it matches this query pattern while producing the least amount of tokens. 

rightEdgeGrams could be useful for search scenarios where you need to see suffix patterns like `ing` or `ver` (replace the leading wildcard access pattern).

nGrmas can be useful if you ALSO want to match with substrings within a token. In the table example, we have the token `nives`.

## Exercise
Continuing to work off of the cluster we've already built with the dataset we've already loaded in the previous exercises, we're going to create a new index for autocomplete.

### Creating the Autocomplete Index

We could update our existing default index defintion from earlier exercises, but for the purpose of just comparing the size difference between these indexes I am going to have us build out a new autocomplete specific index from scratch. 

1. Create a new index on the `sample_mflix.movies` namespace titled `autocomplete`. 

      <img src="/images/AtlasSearch/5-autocomplete/namespace.png" style="height: 40%; width:40%;"/>
      
2. Refine the index. We will want to get a bit more granular than accepting the default `dynamic` index settings provided by Atlas out of the box.

      <img src="/images/AtlasSearch/5-autocomplete/field-mappings.png" style="height: 70%; width:70%;"/>
      
3. Modify the field mapping definition to be of type `autocomplete` against the `title` field. Sinc this field is a string, we can turn off the dynamic toggle.
  - maxGrams: 10
  - minGrams: 3
  - tokenization: edgeGram
  - foldDiacritics: true -- *note* foldDiacritics allows us to `fold` the characters that have special accents below, above or through the character. (ie. č would be mapped to c)
  
      <img src="/images/AtlasSearch/5-autocomplete/definition.png" style="height: 50%; width:50%;"/>
      
4. Save the changes and create the index. Once the index is created, note the size difference between autocomplete and the default, dynamic index we created in the previous exercises. 

      <img src="/images/AtlasSearch/5-autocomplete/size.png" style="height: 20%; width:20%;"/>
      
      > our autocomplete index was against the title field alone while the our default index was indexing every field and subfield(s) within our documents.
      Since we set our minGrams to 3 and our maxGrams to 10, we will have many more tokens on the title field which is where this increase in 
      size is coming from.

### Creating the Autocomplete Aggregation

1. Navigate to the sample_mflix.movies namespace in the collections tab and then click on `aggregation`

    The first stage will be for `$search`. We'll define the index we intend on utilizing as well as change the `text` operator to `autocomplete`.
    We're going to search for the movie batman. Mind that since we specified on minGrams to be 3, the minimum token size we can match against will 
    be of size 3. So if I were to start typing `batman`, autocomplete wouldn't match against any tokens until I type in the first 3 keystrokes. 
  
    Let's type in `bat` and see what comes up. 
  
     ```json
      {
        "$search":{
          "index": "autocomplete",
          "autocomplete": {
            "query": "bat",
            "path": "title"
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
        "score": { "$meta": "searchScore" }
      }
    }
    ```

    <img src="/images/AtlasSearch/5-autocomplete/projectStage.png" style="height: 40%; width:40%;"/>
  
    Here you'll see we do match for the batman movie, but we also match against a lot of battle movies as well. This is the expected behavior. 
    
    <img src="/images/AtlasSearch/5-autocomplete/results.png" style="height: 60%; width:60%;"/>

-------------------------------------------------------------------------------------------------------------------------------------------------------
  
***NOTE***: If you want to see the result set change, revisit the `$search` aggregation and add in an m at the end of the query to get `batm`. This will narrow down your result set even further, returning **ONLY** movies about batman.
  
```json
 {
   "$search":{
     "index": "autocomplete",
     "autocomplete": {
       "query": "bat",
       "path": "title"
     }
   }
 }
```
