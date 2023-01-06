# Vector Search
A sub repository for learning the basics about Embeddings (Vectors) and Vector Search

## What is Vector Search?
Vector Search is a means to search over unstructured data, whether that be text or images by leveraging Machine Learning. Developers will capture context of this unstructured data and represent them numerically as vectors. Given vectors, we can then determine which vectors are numerically similar and thus contextually similar. 

## Why is Vector Search important
Vector Search is important for the same reason full text search is important. It allows us to enahnce an end-user's experience by bringing the data they are searching for closer to them. Reflect on how many times you've needed to search for something you didn't quite know the word of, but you knew it was a mammal with wings, nocturnal and made a "hoot" sound. Without typing in the keywords "bird" or "owl" you may still have hard time finding relevant search results. Vector search overcomes this limitation, allowing you to search by what you mean. It can quickly deliver answers to queries based on context.

## How does this work? 
Vector search engines use distance between embeddings to represent similarity. The problem for searching for related content, mathematically, turns into a search for the nearest neighbors of your query. 

Given the diagram below you can see that we transform our unstructured data on the left into vector representation. At query time, we transform our query into a vector representation and determine what data is most similar to the query by measuring the distance bewtween the vectors. 

![](/images/VectorSearch/HL-Vector-Diagram.png)

If documents are alike, their vectors will be similar. Nearest neighbor algorithms like euclidean distance, cosine similarity, dot product, etc. can all be used to efficiently calcuate the distances between the query vector and the result set vectors to generate similarity scores. 

------------------------------------------------------------------------------------------------------------------------------------------------------

There are a couple different ways these vectors can be represented. Check out the table below for descriptions of what a Sparse Vector is and what a Dense Vector is. 

| # | Label                                                       | Description |
|:--|:------------------------------------------------------------|:-----------|
| 1 | [Sparse Vectors](SparseVectors)  | How do Sparse Vectors introduce semantics to Full Text Search? |
| 2 | [Dense Vectors](DenseVectors)     | How do we make Sparse Vectors more efficient? |
| 3 | [MongoDB Atlas Vector Search](foundations/)     | How does Atlas Search take advantage of Vectors?|
