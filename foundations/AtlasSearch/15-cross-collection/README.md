# Cross Collection Search in MongoDB Atlas Search

So far we’ve been searching against documents within a single collection. This may satisfy plenty of use cases, but what if we have a data model 
that requires that aspects of documents be split up across different collections? How might we be able to keep this type of data model and split our 
search across collections? 

### There are a handful of ways to tackle this problem with Atlas Search today. 
1. Join your collections together, materialize the view, and [perform a search on the materialized view](https://www.mongodb.com/docs/atlas/atlas-search/tutorial/cross-collection-indexes-and-queries/#std-label-cross-collection-indexes-and-queries). 
2. [Embed your search within the $lookup operator](https://www.mongodb.com/docs/atlas/atlas-search/tutorial/lookup-with-search/#std-label-lookup-with-search-tutorial) (starting in v6.0)
3. [Embed your search within the $unionWith operator](https://www.mongodb.com/docs/atlas/atlas-search/tutorial/search-with-unionwith/#std-label-search-with-unionwith-tutorial) (starting in v6.0)

There are some caveats when performing these options. 
1. When searching against a materialized view, your data is only as real time as the view itself. Additionally, you are effectively duplicating the dataset you are looking to search against. 
2. `$lookup` queries are not very performant because Atlas Search does a full document lookup on the database for each document in the collection. For the best performance, you’ll want to reduce the number of lookups performed as much as possible. 


As we progress with more mature versions of MongoDB, I foresee options 2 and 3 of cross collection search being utilized more than option 1. 
Because of that these next two exercises will demonstrate how to build those out.

## Cross Collection Search with $lookup

1. Using the sample dataset, create a default search index against the `sample_analytics.accounts` namespace.

<img src="/images/AtlasSearch/15-cross-collection/lookupindex.png" style="height: 50%; width:50%;"/>

2. Navigate to the `collections` tab and then to the `sample_analytics.customers` namespace and create an aggregation. 
3. Build out the aggregation. We’ll want to join data into the customers collection from the accounts collection using the account id. The accounts data will exist as objects within an array titled `purchases`.  
    - Within the `$lookup` stage we will also add in a `pipeline` that specifies the pipeline to run on the foreign `accounts` collection. 
    - We want to search for customer accounts that have purchased CurrencyService and InvestmentStock, favoring the ones that have an order limit between 5000 and 10000.
    - At the conclusion of our `$search` stage of the pipeline, we can optionally project out the account’s *_id* field. 

      <img src="/images/AtlasSearch/15-cross-collection/lookup.png" style="height: 50%; width:50%;"/>
      
      ```json
      {
         "$lookup": {
          "from": "accounts",
          "localField": "accounts",
          "foreignField": "account_id",
          "as": "purchases",
          "pipeline": [
           {
            "$search": {
             "compound": {
              "must": [
               {
                "queryString": {
                 "defaultPath": "products",
                 "query": "products: (CurrencyService AND InvestmentStock)"
                }
               }
              ],
              "should": [
               {
                "range": {
                 "path": "limit",
                 "gte": 5000,
                 "lte": 10000
                }
               }
              ]
             }
            }
           },
           {
            "$project": {
             "_id": 0
            }
           }
          ]
         }
        }
      ```
      
      
  4. Evaluate the result set. You'll now see we have the account purchases embedded within the corresponding customer's document, 
     and that each purchase object has `CurrencyService` and `InvestmentStock`.
     
     <img src="/images/AtlasSearch/15-cross-collection/lookupresult.png" style="height: 35%; width:35%;"/>
     
 ----------------------------------------------------------------------------------------------------------------------------------------------------------

## Cross Collection Search with $unionWith

1. Using the sample dataset, create a default search index against the `sample_training.companies` and `sample_training.inspections` namespaces.

***Companies*** 

<img src="/images/AtlasSearch/15-cross-collection/companies.png" style="height: 50%; width:50%;"/>

***Inspections***

<img src="/images/AtlasSearch/15-cross-collection/inspections.png" style="height: 50%; width:50%;"/>

2. Navigate to the `collections` tab and then to the `sample_training.companies` namespace and create an aggregation. 
3. Build out the aggregation. We’ll want to union data into the companies collection from the inspections collection. For the `companies` collection, we'll want to search for companies that include *mobile* in the name. For the `inspections` collection, we'll want to accomplish the same thing, however instead of searching for the term *mobile* on the `name` field, it will be on the `business_name` field. 
    - The first `$search` stage, we'll want to perform the search for the term *mobile* against the `name` path. 
    - So we can keep track of which collection a matched document originated from, we're going to add a `$set` stage and set a new `source` field to *companies*
    - For readability, let's `$project` just the `name` and the `source` fields. 
    - Now that we're done with the companies data, let's perform the `$unionWith` operation against the inspections collection. We'll embed the `pipeline` operator of the stage with a `$search` for *Mobile* against the path *business_name*, then we'll `$set` the `source` field to *inspections*, and then lastly we'll project out `name` and `business_name` for readability. 


        <img src="/images/AtlasSearch/15-cross-collection/unionSearchStage.png" style="height: 50%; width:50%;"/>
        <img src="/images/AtlasSearch/15-cross-collection/unionSetStage.png" style="height: 50%; width:50%;"/>
        <img src="/images/AtlasSearch/15-cross-collection/unionProjectStage.png" style="height: 50%; width:50%;"/>
        <img src="/images/AtlasSearch/15-cross-collection/unionWithStage.png" style="height: 50%; width:50%;"/>



        ```json
        [{
         "$search": {
          "index": "default",
          "text": {
           "query": "Mobile",
           "path": "name"
          }
         }
        }, {
         "$set": {
          "source": "companies"
         }
        }, {
         "$project": {
          "_id": 0,
          "name": 1,
          "source": 1
         }
        }, {
         "$unionWith": {
          "coll": "inspections",
          "pipeline": [
           {
            "$search": {
             "text": {
              "query": "Mobile",
              "path": "business_name"
             }
            }
           },
           {
            "$set": {
             "source": "inspections"
            }
           },
           {
            "$project": {
             "_id": 0,
             "business_name": 1,
             "source": 1
            }
           }
          ]
         }
        }]
        ```

4. Evaluate the result set by adding a final match stage and see what comes up when you match against `source`: *comapnies* vs `source`: *inspections*. You should see that we have matches for both. 
