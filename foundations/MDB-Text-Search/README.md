## How does the MongoDB Text Index work?
A MongoDB Text Index will tokenize and stem the content of the field that is being indexed, breaking the string into individual works or tokens that will be further reduced to their stems. These indexes can grow very large. They contain one index entry for each unique, post-stemmed word in every indexed field for every document. 
> for example, talk, talks, talked, talking will all be reduced to their stem "talk"

- Within the text index you can specify a language filter to define the token stems and to avoid indexing of stop words.
- You can also leverage the use of language fields within a document to specify the language filter to use. Alternatively you can specify the language dynamically using the "language_override" key in the index definition.
  > `{ quote : "original" }` to index the document `{_id:1, language:”portugues”, original:”A sorte protege os audazes.”}` with the portuguese language filter. 

  > `{ quote : "text" }, {language_override: "idioma"}` to index the document `{idioma: "english", quote: "is this a dagger which I see before me" }` with a dynamic language filter.

- You can also control the results of the text search by adding weights to particular fields, effectively boosting their relevancy to the query when matched. 
  > `{content: “text”, keywords: “text”, about: “text”}, {weights:{content:10, keywords:5}}`
  
### Pitfalls of MongoDB Text Indexes
1. Can grow to be quite large and demanding of memory
2. Indexes cannot store phrases or information about proximity of words
3. Can only have 1 text index per collection
4. Not effective with case insensitivity searches. Will need to utilize patterns with $regex to achieve case insensitive matches

### $text and $regex for text searches
- **$text** : The $text operator can search for words and phrases. Query matches on the complete stemmed words.
  - [$text demo]()
- **$regex** : The $regex operator can search for patterns and wildcard searches. It can be used with both regular indexes as well as text indexes. *This operator can be very CPU intensive and you lose the ability to score documents*.
  - [$regex demo]()
