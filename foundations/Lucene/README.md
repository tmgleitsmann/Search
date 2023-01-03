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
4. Ancillary Modules (will be exempting from documentation)

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

*Indexing Process Visualized*

![](/images/Lucene/IndexingProcess.png)

- These segments are IMMUTABLE. This will be important later when covering CRUD and segmentation merging. 
- One or many segments can comprise an inverted index.
  - An inverted index is essentially a hash table data strcture that maps content to document location. This is traditionally the other way around, hence the term "inverted" index.

*visualize the difference below*

![](/images/Lucene/invertedIndex.png)

### Searching
Against Lucene a user has the ability to search, filter, paginate and sort from a result set. This is done by breaking down Lucene search into subcomponents.
- Query Model
- Query Evaluation
- Scoring
- Search Extensions


**Query Model**:
Lucene does not enforce a query language, but instead uses query objects to perform searches. These objects are provided as building blocks to express complex queries and can be constructed programmatically or through the *query parser*

The Query Types can be
- Term Queries: Evaluate a single term in a specific field
- Boolean Queries: A Query that matches documents matching boolean combinations of other queries. AND/OR/NOT evaluation
- Proximity Queries: Finding words that are a specified distance away from each other
- Position Based Queries: allow to express more complex rules for proximity and relative positions of terms
- Wildcard: Implements the wildcard search query
- Fuzzy Matching: Implements the fuzzy search query. The similarity measurement is based on the Damerau-Levenshtein (optimal string alignment) algorithm
- Regex: A regular expression based query
- Disjunction-Max Queries: A query that generates the union of documents produced by its subqueries, and that scores each document with the maximum score for that document as produced by any subquery, plus a tie breaking increment for any additional matching subqueries
fields
- etc.

These queries allow for developers to express complex criteria for matching and scoring documents in a well strctured "tree" of query clauses. The QueryParser will typically parse a search into a query tree if the queries weren't generated and combined programatically already.

The QueryTree is an object that represents complex query structures
- Leaf nodes will be your terms
- Edges will be your boolean operators (should, must, filter, etc)
- Non-leaf nodes are the remainder of the boolean query

A visual example can be found below
![](/images/Lucene/queryTree.png)

**Query Evaluation**:
When a query is executed, each inverted index segment is processed sequentially and will generate a Scorer object. Scoreres typically score documents with a *document at a time* strategy but *term at a time* also exists. 

Scorers will move up the tree until they are consolidated by a Collector object that consumes the scores and computes the results. For example, if we wanted to match the top 10 most relevant documents, our Collector will populate the documents within a priority queue of size 10, ranked by score. Collectors can also be used for faceting, grouping, and more. 

**Similarity**: The logic for scoring terms is implemented by the Similary Class in Lucene. Similarity will determine
- Field normalization factors (weights)
  - Depends on field length
  - Depends on user-specified field boosts
- Calculate a score from the scoring model - Some normalization of TD-IDF
  - BM25 (Lucene 4)


-----------------------------------------------------------------------------------------------------------------------------------------

## CRUD Within Lucene
What happens when creating, reading, updating and deleted documents from an inverted index?

- Insert/Update: The original document will be analyzed on the fields to be indexed in accordance with the `Lucene Mapping`. The document is then persisted within a segment that exists on an index. **Note: Segments are IMMUTABLE**. Whenever an update occurs, Lucene will retrieve the document, perform the update and then index the NEW document while [marking the previous document for deletion](https://www.elastic.co/blog/lucenes-handling-of-deleted-documents)?.
    
- Read: Client query will be passed through a query parser and then text analysis chain before being constructed into a QueryTree. The QueryTree will then be executed against the inverted index. Lucene will retrieve the documents hit as well as the corresponding scores constructed. 
    
- Delete: [Lucene will mark the document for deletion](https://www.elastic.co/blog/lucenes-handling-of-deleted-documents)?
    
  - What is a Lucene Mapping?
    - A Lucene Mapping is the definition for the document structure and how it should be indexed.
    - It can be considered the schema for indexing documents

Since segments are immutable, how do we reclaim disk for data that is stale or marked for deletion? **Segmentation Merging** is the process of reclaiming bytes on disk that were previously occupied by documents marked for deletion. This is ultimately when a document in lucene will get deleted. 
*Click on the thumbnail below to see a video on how segments merge while indexing.*

[![Segment Merging](/images/Lucene/segmentMerge.png)](https://www.youtube.com/watch?v=YW0bOvLp72E)


-----------------------------------------------------------------------------------------------------------------------------------------

## Search Foundations / Features

| # | Label                                                       | Description |
|:--|:------------------------------------------------------------|:-----------|
| 1 | [Basic Text Search](foundations/)     | How does text search work today using text indexes and $text/ $regex in MongoDB? |
| 2 | [Fuzzy Text Search](foundations/)     | How can we make our text indexes smarter and more efficient?|
| 3 | [Highlighting](foundations/)          | How does Atlas Search provide full text search capabilities? |
| 4 | [Autocomplete](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 5 | [Keyword](foundations/)               | What is Vector Search and how does it improve on Full Text Search? |
| 6 | [Phrase](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 7 | [Diacritics](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 8 | [Compound](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 9 | [Explain](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 10| [Custom Scoring](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 11| [Faceting](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 12| [Synonyms](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 13| [Custom Scoring](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 14| [Sorting and Pagination](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 15| [Cross-Collection Search](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 16| [Custom Analyzers](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
| 17| [Flexible Querying / Index Intersection](foundations/)          | What is Vector Search and how does it improve on Full Text Search? |
