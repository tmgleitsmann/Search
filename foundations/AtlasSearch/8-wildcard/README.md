# Wildcard Searches in MongoDB Atlas Search
Wildcard is a term-level operator, meaning that the query field itself is not analyzed (remember that traditionally, both the documents and the query are analyzed). 

Because of this, wildcard searches work well with keyword analyzers because the query field matches the exact, original corpus rather than an analyzed one. 

Recall that with the keyword analyzer the following sequence would be analyzed as such

> Example: `The quick brown fox jumped over the lazy dog` would tokenize to `[The quick brown fox jumped over the lazy dog]` rather than traditionally `[quick, brown, fox, jump, over, laz, dog]`

A single token, without alteration. So if we were to run a query for `*brown fox*`, we’d expect to match against a token that contains the text `brown` followed by a `\w` (whitespace) followed by `fox`. 

> How do you imagine this query would work against documents that are indexed using the `standard` analyzer? It wouldn’t. We would be searching for the token `*brown fox*`, but that wouldn’t exist within a single token. 

## Exercise
In this exercise we're going to leverage the same `sample_mflix.movies` dataset, except instead of analyzing the `cast` field, we'll be analyzing the `title` field. 

### Creating the Keyword Index

1. Create a new index on the `sample_mflix.movies` namespace titled `wildcard`. 

      <img src="/images/AtlasSearch/8-wildcard/index-config1.png" style="height: 40%; width:40%;"/>
      
2. Refine the index. We will want to get a bit more granular than accepting the default `dynamic` index settings provided by Atlas out of the box. Plus, we'll want to utilize the keyword lucene analyzer rather than the default.

      <img src="/images/AtlasSearch/8-wildcard/refine1.png" style="height: 70%; width:70%;"/>
      
3. Modify the field mapping definition to be of type `string` against the `title` field. Since this field is a string, we can turn off the dynamic toggle.
  - Change the index and search analyzer to `keyword`
  - Index Options: `docs` --> specifies the amount of information to store for the indexed field.
    - `docs` --> only indexed documents
    - `freqs` --> only indexed documents and term frequency
    - `positions` --> only indexed documents, term frequency and term positions
    - `offsets` --> index documents, term frequency, term positions and term offset
  - Store: `false` --> lucene allows you to store the exact document's text as well as what's analyzed. Since the `keyword analyzer` doesn't change the           text, this would be redundant to have set as true. 
  - Norms: `false` --> String that specifies whether to include or omit the field length in the result when scoring. The field length is used as a               tiebreaker in the case that documents have matching scores. The shorter field length will return first.
  
      <img src="/images/AtlasSearch/8-wildcard/field-mappings2.png" style="height: 50%; width:50%;"/>

### Creating the Wildcard Aggregation

1. Navigate to the sample_mflix.movies namespace in the collections tab and then click on `aggregation`

    The first stage will be for `$search`. We'll define the `keyword` index we intend on using.
    We're going to search for the movie titles containing either the words `man`, `men`, `woman`, or `women`.

    ```json
    {
      "$search": {
        "index": "wildcard",
        "wildcard": {
          "query": "*m?n*",
          "path": "title"
        }
      }
    }
    ```
    <img src="/images/AtlasSearch/8-wildcard/searchStage.png" style="height: 40%; width:40%;"/>

2. The next stage we'll use is `$project` to cleanup our payload. 

      
    ```json
      {
        "$project":{
          "_id":0,
          "title":1
        }
      }
    ```
    
    
    <img src="/images/AtlasSearch/8-wildcard/projectStage.png" style="height: 40%; width:40%;"/>
  
    Here you'll see we are matching for movies titled with the words `men`, `man`, `women` or `woman`. 
    
    <img src="/images/AtlasSearch/8-wildcard/results.png" style="height: 60%; width:60%;"/>
    
-------------------------------------------------------------------------------------------------------------------------------------------------------

***NOTE***: Think about the scoring for a search like this. The title is just a string, so analyzing each title using the keyword analyzer will result in a single token that we need to match against. Recall that the TF-IDF scoring model requires searched term-frequency in a document, the number of total documents and the number of documents containing our searched term. Wouldn't these all be the same across all matching documents?

