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

1. With an Atlas cluster already provisioned, load in the sample data in this repository.

    Navigate inside the data directory and run the following command. This will concatenate all the json files before before performing a mongoimport into 
    the `vector_search_db.responses` namespace. We'll be loading in sample "talk" data where the contents have already been converted into a vector. 

    ```shell
    cat *.json | mongoimport --uri "{clusterURL}" -u {user} -p {password} --db vector_search_db --collection responses
    ```

    The documents you load should all look more or less like this. Note the `vector` field contains *384 embeddings*. 

    <img src="/images/AtlasSearch/17-flexible-querying-index-intersection/result.png" style="height: 50%; width:50%;"/>
  
2. Now that our data is successfully loaded, let's create the Search Index. 

    Navigate to the *Search* tab and create an index using the JSON Editor. We will want to create an index against that vector field with
      - dimensions: 384 (this is representative of the number of embeddings within our vector array)
      - similarity: we'll use cosine similarity for this example
      - type: knnVector 


    ```json
      {
        "mappings": {
          "fields": {
            "vector": [
              {
                "dimensions": 384,
                "similarity": "cosine",
                "type": "knnVector"
              }
            ]
          }
        }
      }
    ```
    
    <img src="/images/AtlasSearch/17-flexible-querying-index-intersection/result.png" style="height: 50%; width:50%;"/>
    
3. The last thing to do is query against our data. 
    Now since we are using vector search, we'll need to convert our query into a vector. Since I'm not providing you with the means to do so in this    README, I'll just provide you the sample query to keep things simple. 
    
    

[Hierachical Navigable Small Worlds Resource](https://www.pinecone.io/learn/hnsw/)
