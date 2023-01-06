# Search
A one-stop shop for learning everything about Search technologies and how they pertain to MongoDB

The purpose behind this repository is to teach everything a Solutions Architect would need to know about Text Search in order to educate their customers on the capabilities of what MongoDB Atlas Search can provide. Here you can learn about MongoDB native text search and why it is becoming obsolete, full text search and why it is becoming the industry standard for quickly retrieving data in a flexible manner, Lucene: the engine behind full text search, different full text search datastores that MongoDB will frequently compete against, and MongoDB Atlas: MongoDB's solution to a fully managed full text search solution.

## Table of Contents
| # | Label                                                       | Description |
|:--|:------------------------------------------------------------|:-----------|
| 1 | [Native MongoDB Text Search](foundations/MDB-Text-Search/)  | How does text search work today using text indexes and $text/ $regex in MongoDB? |
| 2 | [What is Full Text Search - Lucene](foundations/Lucene)     | How can we make our text indexes smarter and more efficient?|
| 3 | [MongoDB Atlas Search Fundamentals](foundations/AtlasSearch)     | How does Atlas Search provide full text search capabilities? |
| 4 | [Vector Search Fundamentals](foundations/VectorSearch)     | What is Vector Search and how does it improve on Full Text Search? |


## Demos
MongoDB Atlas Search can satisfy a variety of different Search Use Cases. Ones that are quickly and easily demo-able you can find below

| # | Demo                                         | Repo         | Description |
|:--|:---------------------------------------------|:-------------|:-----------|
| 1 | [MongoDB Atlas Player Search](https://www.atlassearchsoccer.com/)  | [Github](https://github.com/mongodb-developer/atlas-search-soccer)| Demonstrate how using Full Text Search, Fuzzy Matching, Autocomplete and Faceting can bring complicated player names closer to your end user |
| 2 | [MongoRX](https://mongorx.mside.app/#/dashboard)  | [Github](https://github.com/mongodb-developer/MongoRx)| Combining Atlas Search with App Services, Charts and cloud ecosystem services to build a data discovery & exploration application around medical trials and drugs|
| 3 | [MongoDB Atlas Restaurant Finder](http://atlassearchrestaurants.com/)  | [Github](https://github.com/mongodb-developer/WhatsCooking)   | Demonstrate Full Text Search and Faceting to query against geo-data |
| 4 | Video Vector Search | [Github](https://github.com/wbleonard/atlas-vector-search-video/tree/main) | Prototype of how Atlas Vector Search could be used to find videos with relevant content - requiring that no descriptive metadata be stored along with the videos. |

