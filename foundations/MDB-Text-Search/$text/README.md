# Text Search Demo using $text

### 1. First thing we will want to do is create an Atlas Cluster
Any config should do. I'll be building out a Single Region M10 on AWS.

![Atlas Config](/images/config1.png)

### 2. Let's load in some sample data
While the sample data loads in be sure to configure network security and RBAC.

![Load in sample data](/images/dataload.png)

### 3. Create The Text Index on the `sample.mflix` namespace
I will be using the shell in order to run the `createIndex` command
> `db.movies.createIndex({"fullplot":"text", "title":"text"}, {weights:{fullplot:2, title:5}, name:"TomTextIndex"})`

This index allows us to search against both the fullplot and title fields of the movies collection. Matches for tokens against fullplot will be 2x more relevant than a default scored token and matches for tokens against title will be 5x more relevant. 

![Creating the Index in Shell](/images/textindex.png)

### 4. Querying against our Text Index
Run the following command against the `movies` collection
> `db.movies.aggregate([ { $match: { $text: { $search: "anvil" } } }, { $project: { score: { $meta: "textScore" }, title: 1, fullplot: 1 } }, { $sort: { score: -1 } } ])`

You should get 3 documents back, each having the title and/or fullplot corresponding to the search term "anvil". Note the score of the documents. 

![Search using $text](/images/text.png)
