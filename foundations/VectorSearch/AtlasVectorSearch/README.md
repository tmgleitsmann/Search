# Vector Search in MongoDB Atlas Search

In efforts to align with the needs of developers to provide dense vector search capabilities within their applications, MongoDB Atlas allows you to store 
and search against vector embeddings. Through approximate nearest neighbor searching, we can now search for documents that are most similar to a query not 
based on TF-IDF, but how close together they are on a vector plane. 

MongoDB Atlas Search uses the [*Hierarchical Navigable Small World algorithm*](https://arxiv.org/abs/1603.09320) to perform these types of searches.
I will also include a separate link at the bottom of this repository 

Atlas Search uses the `knnBeta` operator to perform vector search. It has the following configurable options:
- filter: Any Atlas Search operator that can filter documents to narrow down the scope of the vector search
- k: Number of nearest neighbors to return
- path: The indexed kNN vector field to search against
- vector: The array of numbers that represent the query vector


# Exercise




[Hierachical Navigable Small Worlds Resource](https://www.pinecone.io/learn/hnsw/)
