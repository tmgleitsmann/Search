# Faceting in MongoDB Atlas Search

Facets are a subset of filtering. They allow you as a developer to categorize your result set so that visitors can quickly refine their options 
without scrolling through pages of irrelevant results in search for something particular. 

Here is an example of faceting on an application I'm sure we've all used at one point or another... Amazon. 

<img src="/images/AtlasSearch/12-facet/facet_ex.png" style="height: 20%; width:20%;"/>

Categorize results? Refining options? Isn't this what filtering is? 

Almost... filters are typically defined by the user at query time to narrow down the wholistic result set. Lucene will handle the querying against filters to return 
a more preliminary result set. From there, the user can select facets defined by the developer to further narrow the result set. Filters are handled by MongoDB in this case. Facets will be handled on the application end within the UI. 

## Faceting Example 

You can use facet with both the `$search` and `$searchMeta` stages. MongoDB recommends using facet with the `$searchMeta` stage to retrieve metadata results only for the query.
- facet: groups results by values or ranges in the specified faceted fields and returns the count for each of those groups.
- operator: defines the operator used to perform the facet over.
- facets: corresponding groups we will be defining as buckets for the facet. You must define at least 1 `facets`.

```json
{
  "$searchMeta": {
    "facet":{
      "operator": {
        "<operator-specification>"
      },
      "facets": {
        "<facet-name>" : {
          "type" : "string",
          "path" : "<field-path>",
          "numBuckets" : "<number-of-categories>",
        }
      }
    }
  }
}
```

### Faceting Exercise

Attached to this repository is a `players2020.bson` file. It contains the Fifa 20 players dataset that we will want to load into a cluster before building out our faceted search. 

1. Download and `mongorestore` the dataset. 

```console
mongorestore --drop -d players2020 -c players <Path/To/Dataset> --uri mongodb+srv://<username>:<password>@search.yimh2.mongodb.net
```
2. Once our dataset is loaded, lets create a search index for it. We'll need to include index definitions for the fields we want to facet against as well as the fields we want to search against. 

> We will want to search against the player name and filter against the position, club, country and overall rating for the player. 
> Additionally, we will want to facet against the club, country and position. 


<img src="/images/AtlasSearch/12-facet/index1.png" style="height: 50%; width:50%;"/>

Create a default `String` mapping against
- Name
- Club
- Country
- Position ( NOT Positions :) )


<img src="/images/AtlasSearch/12-facet/Club.png" style="height: 50%; width:50%;"/>

Create an `Int64` mapping against 
- Overall

<img src="/images/AtlasSearch/12-facet/Overall.png" style="height: 50%; width:50%;"/>

Create `StringFacet` mapping against
- Club
- Country
- Position

<img src="/images/AtlasSearch/12-facet/ClubFacet.png" style="height: 50%; width:50%;"/>

This is an index definition that will allow us to query against Name, Club, Country and Position, and Facet against Club, Country and Position.

<img src="/images/AtlasSearch/12-facet/index2.png" style="height: 50%; width:50%;"/>


3. Navigate to the `Aggregation` tab within `Collections>Players2020>Players` namespace in the Atlas UI. Lets build out the `$searchMeta` aggregation.

<img src="/images/AtlasSearch/12-facet/searchMeta.png" style="height: 35%; width:35%;"/>

You'll see within the `operator` operator we are defining how we want to QUERY the players. This will look no different than how we would define a standard `search` aggregation. 

Once we get down to the `facets` operator however, we'll want to define how/where we categorize the players we just queried for. In this example, we will be creating buckets for `country`, `club`, and `position` where they each will have a defined maximum number of buckets. 

```json
{
  "index": "faceted_search",
  "facet": {
    "operator": {
      "compound":{
        "must":[
          {
            "range":{
              "path":"Overall",
              "gte":80
            }
          }
        ],
        "filter":[
          {
            "text":{
              "query":"Arsenal",
              "path":"Club"
            }
          }
        ]
      }
    },
    "facets": {
      "countryFacet": {
        "type": "string",
        "path": "Country",
        "numBuckets": 100
      },
      "clubFacet": {
        "type": "string",
        "path": "Club",
        "numBuckets": 100
      },
      "positionFacet":{
        "type":"string",
        "path":"Position",
        "numBuckets":10
      }
    }
  }
}
```


In the output of this single aggreagtion stage you will see `count` and `facet` objects. 
- count: Object containing the number of documents matched against in the result set. 
- facet: Each facet result document contains the buckets option, which is an array of resulting buckets for the facet. Each resulting bucket contains the number of documents that correspond to that *category* from the result set. 

An example of what this looks like would be

<img src="/images/AtlasSearch/12-facet/searchMetaResults.png" style="height: 35%; width:35%;"/>


----------------------------------------------------------------------------------------------------------------------------------------------------------

In this example we only had `StringFacets` to work with, however you can also create buckets with `NumericFacets` and `DateFacets`. With these facets you can leverage a different syntax to more dynamically create your buckets. 

```json
{
  "$searchMeta": {
    "facet":{
      "operator": {
        "<operator-specification>"
      },
      "facets": {
        "<facet-name>" : {
          "type" : "date | number",
          "path" : "<field-path>",
          "boundaries" : "<array-of-dates> | <array-of-numbers>",
          "default": "<bucket-name>"
        }
      }
    }
  }
}
```
