## [GloVe](https://nlp.stanford.edu/pubs/glove.pdf)

GloVe (Global Vectors) is very similar to Word2Vec in the sense that you’re provided with context-independent embeddings. However, the means of getting these embeddings 
is different. In Word2Vec, we were only considering the local context of the dataset, as described with CBOW and Skip Gram. 
With GloVe, we introduce the idea of `global` context for words

So provided global context, we can determine the relationships between words as P(k|i) or probability of k given i. 
- example: What is the probability of a word appearing in the context of another ***across our entire dataset*** (global context).

  |     *probabilities*                 | k=solid | k=gas | k=water | k= *random* |
  |:------------------------------------|:--------|:------|:--------|:------------|
  | P( k \| "ice" )                     | high    | low   | high    | low         |
  | P( k \| "steam" )                   | low     | high  | high    | low         |
  | P( k \| "ice" ) / P( k \| "steam" ) | >1      | <1    | ~1      | ~1          |
  
  >large values (much greater than 1) correlate well with properties specific to ice and small values (much less than 1) correlate well with properties specific to steam.
