# What is Full Text Search?
Measure of accuracy between the relationship of the search query and the result set. 
This might be difficult to accomplish and expensive through pattern matching against traditional indexes. 
To provide developers with fast relevance based searches, an open source search library called Lucene from Apache was created. 

## What is [Lucene](http://opensearchlab.otago.ac.nz/paper_10.pdf)?
Apache Lucene is a modern open source search library designed to provide relevant search capabilities paired with high performance.

Lucene has four main components
1. Analysis of Incoming Content and Queries
2. Indexing and Storage
3. Searching
4. Ancillary Modules

Architecture Diagram

![](/images/Lucene/architecture.png)


### Analysis of Incoming Content
The first section of the diagram that documents and queries get routed to is the Analysis Chain. It can be comprised of 3 different parts.
1. Character Filters: Adds, removes or changes characters from an incoming stream of characters
2. Tokenizers: A Lucene TokenStream object that splits the character stream into tokens
3. Token Filters: Abstract Base Class of TokenStream that removes certain tokens

The output of the analysis chain will be a token stream that flows into an index writer. This is where our "tokens" will be "inverted" and stored on segments in memory.

### Index and Storage
Documents that get passed into Lucene are remodeled to be a flat, ordered list of fields with content. Fields will have the follow attributed to them:
- Name 
- Content Data
- Float Weight (used for scoring)
- Other Attributes (depending on their type)

This combination of fields will determine how the document is processed and represented within a Lucene Index.

Lucene has two broad categories of fields that are not mutually exclusive.
- One that carries content to be inverted (indexed)
  - ie. `{_id:1, "content":"The quick brown fox"}` gets inverted into `{"quick":[1], "brown":[1], "fox":[1]}`
- The other that carries content to be stored as is (stored-source)
  - Stores data "as-is" since storing per-document data can be cumbersome to obtain through a separate system.
  - You can think of stored-source as caching other fields of a document alongside the index to have the same benefits as a covered query in MongoDB. 

Once the token stream from the analysis chain is processed by the indexing chain where it gets inverted, it then gets stored within an in-memory "segment".
- These segments are IMMUTABLE. This will be important later when covering CRUD and segmentation merging. 
- One or many segments can comprise an inverted index.
  - An inverted index is essentially a hash table data strcture that maps content to document location. This is traditionally the other way around, hence the term "inverted" index.
