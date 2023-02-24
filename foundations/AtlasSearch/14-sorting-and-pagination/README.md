# Sorting and Pagination in MongoDB Atlas Search

This repo is prone to change: The current implementation of pagination is functional but not ideal for particular scenarios where customers need deep pagination at lower latency. 
Faster pagination is something currently [in the works](https://feedback.mongodb.com/forums/924868-atlas-search/suggestions/41075920-faster-pagination). 

Additionally, Lucene based sort is in the works and is in beta currently. Once we have an update I will be sure to update this repository. 
The functionality for sort is pretty self explanatory though. Currently, Lucene returns documents sorted in terms of relevancy, 
however what if we want the relevant documents sorted by a different order? 

----------------------------------------------------------------------------------------------------------------------------------------------------------

How often have you tried searching for something very specific, only for it to be absent from the first page of results… and the second page… and the third page. 
You are going to traverse the relevance-based result set until you find that specific artifact that you were looking for. 
Bam, turns out it was on page 20, ranked as the 413th most relevant result. 

It would be disastrous to send all that data in a single request back to a client, especially when 99% of requests will be satisfied by the first “page” of results. 
But for that 1%, we will need an elegant solution to tell Lucene that we haven’t found what we are looking for and that we want to continue searching 
where we left off. This is the pagination problem. 



There are different ways of tackling this problem. Here are two of the most common types of pagination.
1. Cursor based pagination: Provides a pointer to the area of a result set that can be retrieved next. You can think of this as an iterable object in a programming language.
  - Pros
      - Fast and Easy to Use
      - Can provide a snapshot view of the data, meaning that if data is added or removed, it will not impact the cursor
  - Cons
      - You cannot “jump” to particular pages, data retrieval MUST be sequential.
      - Because of this sequential data retrieval, you cannot parallelize the request from the client
2. Offset based pagination: paginate by skipping a number of documents
  - Pros
      - Easy to Implement
      - Supports parallel client requests
      - Stateless (but this could also be viewed as a con)
  - Cons
      - Performance degradation as the pagination gets deeper

----------------------------------------------------------------------------------------------------------------------------------------------------------

## Can this be done in ElasticSearch?

In elasticsearch, this cursor based pagination can more or less be accomplished using the [scroll api](https://www.elastic.co/guide/en/elasticsearch/reference/current/scroll-api.html) (with some serious exceptions, especially at scale).

In elasticsearch, the offset based pagination can be accomplished using `size` and `from` fields within the request, as if they were $limit and $skip respectively. 
Elasticsearch limits the pagination to no more than 10,000 hits with this approach. Developers can alternatively lean on the [`search_after`](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#search-after) field to scale past this limit.  


## Exercise
With Atlas Search, we are going to demonstrate how to paginate using $skip and $limit. 
