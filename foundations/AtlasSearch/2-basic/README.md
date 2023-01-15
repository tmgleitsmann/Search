# Build Basic Full Text Search in MongoDB Atlas

This demo will show how to get started with taking advantage of the fully managed MongoDB Atlas Search capabilities.

The steps are as follows:
1. Configure the Cluster. This includes setting up network and RBAC as well as loading in sample data
2. Configure the Atlas Search Index
3. Build out our Atlas Search Aggregation

## Configure the Cluster

### 1. First thing we will want to do is create an Atlas Cluster
Any config should do. I'll be building out a Single Region M10 on AWS.

![Atlas Config](/images/config1.png)

### 2. Let's load in some sample data
While the sample data loads in be sure to configure network security and RBAC.

![Load in sample data](/images/dataload.png)

## Configure our Atlas Search Index

### 1. Once the data is loaded, navigate to the Search tab of the cluster and `Create Search Index`

![Search Button](/images/AtlasSearch/2-basic/create-search-index.png)

### 2. When Prompted to use the Visual Editor or JSON Editor, select `Visual Editor`

![Search Editor](/images/AtlasSearch/2-basic/visual-editor.png)

### 3. Leave the name as `default` and select the `sample_mflix.movies` namespace

![Search Namespace](/images/AtlasSearch/2-basic/index-namespace.png)

Leave the default settings for the index and create the index. This should leave you with the following screen.

![searchprogress](/)
