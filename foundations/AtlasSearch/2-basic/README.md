# Build Basic Full Text Search in MongoDB Atlas

This demo will show how to get started with taking advantage of the fully managed MongoDB Atlas Search capabilities.

The steps are as follows:
1. Configure the Cluster. This includes setting up network and RBAC as well as loading in sample data
2. Configure the Atlas Search Index
3. Build out our Atlas Search Aggregation

## Configure the Cluster

### 1. First thing we will want to do is create an Atlas Cluster
Any config should do. I'll be building out a Single Region M10 on AWS.

<img src="/images/config1.png" style="height: 60%; width:60%;"/>

### 2. Let's load in some sample data
While the sample data loads in be sure to configure network security and RBAC.

![Load in sample data](/images/dataload.png)

----------------------------------------------------------------------------------------------------------------------------------------------------

## Configure our Atlas Search Index

### 1. Once the data is loaded, navigate to the Search tab of the cluster and `Create Search Index`

<img src="/images/AtlasSearch/2-basic/create-search-index.png" style="height: 60%; width:60%;"/>

### 2. When Prompted to use the Visual Editor or JSON Editor, select `Visual Editor`

<img src="/images/AtlasSearch/2-basic/visual-editor.png" style="height: 60%; width:60%;"/>

### 3. Leave the name as `default` and select the `sample_mflix.movies` namespace

<img src="/images/AtlasSearch/2-basic/index-namespace.png" style="height: 60%; width:60%;"/>

Leave the default settings for the index and create the index. This should leave you with the following screen.

<img src="/images/AtlasSearch/2-basic/status.png" style="height: 80%; width:80%;"/>

----------------------------------------------------------------------------------------------------------------------------------------------------

## Build out our Atlas Search Aggregation

### At this point we can navigate to the `sample_mflix.movies` namespace and create an aggregation

<img src="/images/AtlasSearch/2-basic/aggregation-1.png" style="height: 75%; width:75%;"/>

1. The first stage will be the `$search` stage. We'll want to search for movies that correspond to `vampires and werewolves` in the `fullplot` field.

We will also be including the `highlight` operator within the `$search` stage to show which tokens our query matches against. 

  ```json
  {
    "$search":{
      "index": "default",
      "text": {
        "query": "werewolves and vampires",
        "path": "fullplot"
      },
      "highlight":{"path":"fullplot"}
    }
  }
  ```
<img src="/images/AtlasSearch/2-basic/search-stage.png" style="height: 40%; width:40%;"/>

2. The second stage will be the `$project` stage. We'll want to only show the title, fullplot, search score, and the highlights

  > Note: In practice we would normally want to add a `$limit` stage after `$search` to limit the result set for a boost in performance.  

  ```json
  {
    "$project":{
      "title":1,
      "fullplot":1,
      "score": { "$meta": "searchScore" },
      "highlight": {"$meta": "searchHighlights" }
    }
  }
  ```
<img src="/images/AtlasSearch/2-basic/project-stage.png" style="height: 40%; width:40%;"/>

You can inspect the result set and see that they come pre-sorted by search relevance. You can further inspect the highlight array field to see which tokens influenced the score. 

<img src="/images/AtlasSearch/2-basic/hits.png" style="height: 40%; width:40%;"/>

----------------------------------------------------------------------------------------------------------------------------------------------------

## Reflection

Up to this point in the repository we have gotten our hands dirty building  our very own, custom full text search engine in Python as well as standing up a competitor full text search engine (ElasticSearch, Solr, etc.). One of MongoDB Atlas Search's greatest strengths is how much of the process it abstracts out for developers, allowing them to dive straight into application development rather than dealing with tedious plumbing and infrastructure challenges. 

