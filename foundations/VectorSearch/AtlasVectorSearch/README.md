# Vector Search in MongoDB Atlas Search

In efforts to align with the needs of developers to provide dense vector search capabilities within their applications, MongoDB Atlas allows you to store 
and search against vector embeddings. Through approximate nearest neighbor searching, we can now search for documents that are most similar to a query not 
based on TF-IDF, but how close together they are on a vector plane. 

MongoDB Atlas Search uses the [*Hierarchical Navigable Small World algorithm*](https://arxiv.org/abs/1603.09320) to perform these types of searches.
I will also include a separate link at the bottom of this repository 

MongoDB will index the vectors as a `knnVector` type. This vector type is expected to be an array of int32, int64, or doubles. The knnVetor type will have the following options:
- type: Label that indentifies this field type. Must be `knnVector`
- dimensions: Number of vector dimensions that MongoDB Atlas enforces at index and query time. Cannot exceed 1024
- similarity: Vector similarity function to use for kNN evaluation.
  - `euclidean` : measures the distance between ends of vectors
  - `cosine`: measures similarity based on the angle between vectors
  - `dotProduct`: measure similarity based on both angle and magnitude


Atlas Search uses the `knnBeta` operator to perform vector search. It has the following configurable options:
- filter: Any Atlas Search operator that can filter documents to narrow down the scope of the vector search
- k: Number of nearest neighbors to return
- path: The indexed kNN vector field to search against
- vector: The array of numbers that represent the query vector


*Note: The magnitude of the vector, in every scenario, should be normalized.*

---------------------------------------------------------------------------------------------------------------------------------------------------------

# Exercise




[Hierachical Navigable Small Worlds Resource](https://www.pinecone.io/learn/hnsw/)
