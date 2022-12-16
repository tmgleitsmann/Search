# Text Search Demo using $text

### 1. First thing we will want to do is create an Atlas Cluster
Any config should do. I'll be building out a Single Region M10 on AWS.

![Atlas Config]()

### 2. Let's load in some sample data
While the sample data loads in be sure to configure network security and RBAC.

![Load in sample data]()

### 3. Create The Text Index on the `sample.mflix` namespace
I will be using the shell in order to run the `createIndex` command
> `db.movies.createIndex({"fullplot":"text", "title":"text"}, {weights:{fullplot:2, title:5}, name:"TomTextIndex"})`
