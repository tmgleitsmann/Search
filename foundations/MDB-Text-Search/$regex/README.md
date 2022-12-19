# Text Search Demo using $regex
If you have already completed the $text demo, we can re-use the cluster and index configuration (from steps 1-3). If not follow these steps.

### 1. First thing we will want to do is create an Atlas Cluster
Any config should do. I'll be building out a Single Region M10 on AWS.

![Atlas Config]()

### 2. Let's load in some sample data
While the sample data loads in be sure to configure network security and RBAC.

![Load in sample data]()

### 3. Create The Text Index on the `sample.mflix` namespace
I will be using the shell in order to run the `createIndex` command
> `db.movies.createIndex({"fullplot":"text", "title":"text"}, {weights:{fullplot:2, title:5}, name:"TomTextIndex"})`

This index allows us to search against both the fullplot and title fields of the movies collection. Matches for tokens against fullplot will be 2x more relevant than a default scored token and matches for tokens against title will be 5x more relevant. 

![Creating the Index in Shell]()

### 4. Querying against our Text Index
Run the following command against the `movies` collection. Note that with `$regex` we are capable of matching against patterns and case-insensitivity, however we lose the ability to score our documents. 
> `db.movies.aggregate([ { $match: { title: { $regex: /^bat.*/, $options: "i" } } }, { $limit: 5 }, { $project: { title: 1 } }])`

You should get a handful of documents back, each beginning with the letters "bat" and then having 0 or more characters as the suffix. 

![Search using $regex]()
