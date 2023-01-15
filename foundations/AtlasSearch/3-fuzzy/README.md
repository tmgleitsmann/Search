# Fuzzy Searching

We've gotten our hands dirty with implementing basic full text search in Atlas Search. But Lucene is capable of much more robust querying. 
Fuzzy Search is the ability to account for text mispellings when searching over a corpus. This means that even if an end-user doesn't know how to spell who or what he's searching for, he/she can still find proper results. 

## How does Fuzzy Search 

Fuzzy Search takes advantage of a popular algorithm used to measure the difference between two strings. It iterates through each character in a token to see if there are alternative tokens it can match to by editing `n` number of characters. A great primer for what Levenshtein's Distance Equation is as well as how it is implemented can be found here: 

[Levenshtein Edit Distance Primer](https://medium.com/@ethannam/understanding-the-levenshtein-distance-equation-for-beginners-c4285a5604f0)

----------------------------------------------------------------------------------------------------------------------------------------------------------

## Getting Started 

Luckily we don't need to change anythign to the index we created in `2-basic`. So if you've completed that section of the respository, keep the setup it left you with. If you've jumped straight here without completing `2-basic` then complete steps 1 and 2 of that directory before continuing here. 

1. Navigate to the `sample_mflix.movies` nameespace and click on the `aggregation` tab. 

   <img src="/images/AtlasSearch/2-basic/aggregation-1.png" style="height: 75%; width:75%;"/>

2. Select `$search` as the first aggregation stage and copy the following JSON snippet into the text editor

  ```json
  {
    "index": "default",
    "text": {
      "query": "might cub",
      "path": "title",
      "fuzzy":{
        "maxEdits":2
      }
    },
    "highlight": {"path": "title"}
  }
  ```
  
Here, we're searching for `fight club` but as you might have noticed, a couple mispellings were made. The `fuzzy` operator by default will account for up to two character misses, however a custom amount can be specified through the `fuzzy.maxEdits` operator. 

Fuzzy allows you to do several things when searching over text
  - `maxEdits`: Maximum number of single-character edits required to match the specified search term. Value can be 1 or 2. The default value is 2. Uses 
Damerau-Levenshtein distance. [MongoDB Documentation](https://www.mongodb.com/docs/atlas/atlas-search/text/)
  - `prefixLength`: Number of characters at the beginning of each term in the result that must exactly match. The default value is 0.
  - `maxExpansions`: The maximum number of variations to generate and search for. This limit applies on a per-token basis. The default value is 50.
  
<img src="/images/AtlasSearch/3-fuzzy/search-stage.png" style="height: 75%; width:75%;"/>
  
3. Select `$project` as the second stage and copy the following JSON snippet into the text editor

```json
  {
    "_id":0,
    "title":1,
    "highlight": {"$meta": "searchHighlights"},
    "score": { "$meta": "searchScore"},
  }
  ```
  
  Here, we're cleaning up our output so you can see exactly which documents we're matching against, and how. 
  
  <img src="/images/AtlasSearch/3-fuzzy/project-stage.png" style="height: 75%; width:75%;"/>
  
  `might cub` is 2 edits away from `fight club` and matches against 2 tokens which is why it produces the highest relevance score. 
  
 ----------------------------------------------------------------------------------------------------------------------------------------------------------
 
 # Takeaway
 
Fuzzy Matching is a great solution to bringing data even closer to your end user. Frequently a user might not know how to spell what he/she is looking for or makes a typo in their search. Rather than showing no/poor results, we can be smart and make inferences on who/what was being search for. 

A great application that demonstrates this is the [Atlas Search Soccer App](https://www.atlassearchsoccer.com/). You can leverage fuzzy matching to search for complicated soccer player names. 


# References
https://medium.com/@ethannam/understanding-the-levenshtein-distance-equation-for-beginners-c4285a5604f0
