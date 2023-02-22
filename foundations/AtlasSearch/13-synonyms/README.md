# Synonyms with MongoDB Atlas Search

I recall when I would spend multiple months abroad at a time that even the simplest of words would escape my vocabulary once I returned home. 
There was one time in particular after coming home to Texas from France where I forgot how to say "trash can". I knew what I wanted to say, I knew how to describe it,
but I didn't know the term anymore. And this can be problematic if I need to match against tokens in a corpus. 

Luckily, Lucene allows us to configure our indexes to map against synonyms as if it had access to a thesaurus. So instead of searching for the tokens "trash can", 
I could alternatively search for "bin" and get the same result set. 

Here is how this logic might look as a finite state machine. Books can be mapped with guides and rocks can be mapped with stones in this example. 

<img src="/images/AtlasSearch/13-synonyms/fsm.png" style="height: 50%; width:50%;"/>

How might this work under the hood though? Are we applying synonyms at index time or at query time? And what are the implications of doing one way over the other?

- Query Time: Query time synonyms expand terms used in a query and are therefore able to match more documents in the index.
- Index Time: Index time synonyms expand the terms used at the point of indexing. The index itself then contains more terms, making documents more easily matchable.

Index Time synonyms may look to be the more appealing option based off of these descriptions, however there are some major caveats. 
- The synonym list cannot be updated without reindexing everything, which is very inefficient in practice.
- The search score would be impacted because synonym tokens are counted as well.
- The indexing process becomes more time-consuming and the indexes will get bigger. It is negligible for small data set but is very significant for big ones.

Query Time synonyms move all the additional complexities to query time. This means that instead of searching for a token, you're searching for that token as well as all of its synonyms by constructed a more complex lucene query under the hood. 
- This additional complexity can add latency to the query. 


MongoDB Atlas Search uses the latter (Query Time). There are 3 major steps to incorporating Synonyms in your Atlas Search Queries. 
1. Configuring the Synonyms against the Index. *this still isn't index time definition*
2. Creating the Synonyms Collection in MongoDB
3. Modifying our Queries to incorporate the Synonyms

<img src="/images/AtlasSearch/13-synonyms/steps.png" style="height: 50%; width:50%;"/>

