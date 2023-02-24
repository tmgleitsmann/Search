# Sorting and Pagination in MongoDB Atlas Search

This repo is prone to change: The current implementation of pagination is functional but not ideal for particular scenarios where customers need deep pagination at lower latency. 
Faster pagination is something currently [in the works](https://feedback.mongodb.com/forums/924868-atlas-search/suggestions/41075920-faster-pagination). 

Additionally, ***Lucene based sort is in the works and is in beta currently.*** Once we have an update I will be sure to update this repository. 
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

To get a preview of the type of functionality we will be building out, navigate to the [AtlasSearchSoccer](https://www.atlassearchsoccer.com/) application. We will be searching for the Portuguese superstar, `Ronaldo`. 

<img src="/images/AtlasSearch/14-sorting-and-pagination/initialResults1.png" style="height: 50%; width:50%;"/>

The Ronaldo that we're looking for doesn't pop up. Now I'd argue that this is also a scoring issue that could be addressed, but this is the default behavior and Lucene is working exactly the way it is expected to work. 

Let's navigate to the second page to by clicking on the second soccer ball icon at the bottom of the list of players and see if we can find Ronaldo on the next page. 

<img src="/images/AtlasSearch/14-sorting-and-pagination/initialResults2.png" style="height: 50%; width:50%;"/>

And here, on the second row and fourth column, we do find Ronaldo. How might I be able to build an aggregation that returns the second page of players?

## 1. Download and `mongorestore` the players dataset. 

*Note:* This is not the same dataset as the AtlasSearchSoccer application.  

```console
mongorestore --drop -d players2020 -c players <Path/To/Dataset> --uri mongodb+srv://<username>:<password>@search.yimh2.mongodb.net
```

## 2. Create a default search index on the players dataset. 

No fancy index required to test out pagination. 

<img src="/images/AtlasSearch/14-sorting-and-pagination/defaultIndex.png" style="height: 50%; width:50%;"/>

## 3. Create our Pagination Aggregation. 

First, Pagination Aggregation would be a sick band name. But in all seriousness it requires 3 stages. 
  1. The $search stage: to perform the search
  2. The $skip stage: to skip over previously visited documents
  3. The $limit stage: to limit our result set to a single *page* of documents
  
Navigate to the `Aggregation` tab within `Collections>Players2020>Players` namespace in the Atlas UI.

**Note:** The player we are looking for is `C. Ronaldo dos Santos Aveiro` with an overall rating of `93`.

<img src="/images/AtlasSearch/14-sorting-and-pagination/searchStage.png" style="height: 50%; width:50%;"/>

```json
{
 "$search": {
  "index": "default",
  "text": {
   "query": "Ronaldo",
   "path": "Name"
  }
 }
}
```

**Note:** The Aggregation Framework in the Atlas UI automatically limits the number of documents returned to 10.

Now you can take my word that the Ronaldo we're looking for isn't in the initial 10 results that the UI provides us, OR you can add a project stage and see for yourself. 

```json
{
  "$project":{
    "Name":1,
    "Overall":1
  }
}
```

So Ronaldo isn't popping up in our first page of 10 players. How can we re-run this search to get the next set of players while ignoring the ones we've already visited? By using `$skip` and `$limit` of course.

<img src="/images/AtlasSearch/14-sorting-and-pagination/skiplimit.png" style="height: 50%; width:50%;"/>

```json
{
 "$skip": 10
}, {
 "$limit": 10
}
```

Check out the results. You can see that the Ronaldo we are looking for exists on the second page. Remember that Lucene will return documents based on their relevance score, so the Ronaldo we are looking for was deemed the least relevant by Lucene. Why might that be? Well, we are matching for 1 of the 5 tokens in Ronaldo's name, while no other player has more than 4 tokens in their name. This is negatively impacting Ronaldo's score. 

<img src="/images/AtlasSearch/14-sorting-and-pagination/results.png" style="height: 50%; width:50%;"/>




