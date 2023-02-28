# Custom Analyzers in MongoDB Atlas Search

Before we get hands on with building custom analyzers in MongoDB Atlas Search, I think it would be useful to revisit what an analyzer is and the pieces 
it is composed of. If you haven’t yet, please read the [Lucene](./Lucene/README.md) portion of the repository, specifically around ***Analysis of Incoming Content***. 
This will describe what the *character filter*, *tokenizer* and *token filter* does. 

------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Character Filters: Atlas Search supports 4 types of character filters
- [htmlStrip](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/character-filters/#std-label-htmlStrip-ref): strips out HTML constructs
- [icuNormalize](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/character-filters/#std-label-icuNormalize-ref): normalizes text with the [ICU filter](https://icu.unicode.org/)
- [mapping](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/character-filters/#std-label-mapping-ref): applies user-specified normalization mappings to characters
- [persian](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/character-filters/#std-label-persian-ref): replaces instances of [zero-width non-joiner](https://en.wikipedia.org/wiki/Zero-width_non-joiner) with the space character


------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Tokenizers: Determines how Atlas Search splits up text into discrete chunks for indexing. 

We’ve had practice up to this point with applying particular analyzers.  Atlas Search supports the following tokenizers. 
- [edgeGram](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/tokenizers/#std-label-edgegram-tokenizer-ref): Tokenizes input from the left side, or "edge", of a text input into n-grams of given sizes. We commonly see this sort of tokenizer used in [***autocomplete*** ](https://github.com/tmgleitsmann/Search/tree/main/foundations/AtlasSearch/5-autocomplete) searches
- [nGram](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/tokenizers/#std-label-ngram-tokenizer-ref): Tokenizes into text chunks, or "n-grams", of given sizes. This can be beneficial when querying languages that don’t use spaces or that have long compound words
- [keyword](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/tokenizers/#std-label-keyword-tokenizer-ref): Tokenizes the entire input as a single token. We commonly see this tokenizer used in [***keyword***](https://github.com/tmgleitsmann/Search/tree/main/foundations/AtlasSearch/6-keyword) searches
- [regexCaptureGroup](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/tokenizers/#std-label-regexcapturegroup-tokenizer-ref): Tokenizes a regular expression pattern to extract tokens. Each matched pattern is tokenized
- [regexSplit](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/tokenizers/#std-label-regexSplit-tokenizer-ref): splits tokens with a regular-expression based delimiter
- [standard](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/tokenizers/#std-label-standard-tokenizer-ref): Tokenizes based on word break rules from the [***Unicode Text Segmentation algorithm***](https://www.unicode.org/L2/L2019/19034-uax29-34-draft.pdf)
- [uaxUrlEmail](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/tokenizers/#std-label-uaxUrlEmail-tokenizer-ref): Tokenizes URLs and email addresses
- [whitespace](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/tokenizers/#std-label-whitespace-tokenizer-ref): Tokenizes based on occurrences of whitespace between words

------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Token Filters: Accept a stream of tokens from a tokenizer and can modify, delete or add tokens to the stream. 

There are a ton of different token filters, 
many of which can be applied in conjunction with one another. There are too many token filters to list but Atlas Search supports at least: 
[asciiFolding](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#asciifolding), 
[daitchMokotoffSoundex](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#daitchmokotoffsoundex), 
[edgeGram](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#edgegram), 
[nGram](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#ngram), 
[englishPossesive](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#englishpossessive), 
[flattenGraph](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#flattengraph), 
[icuFolding](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#icufolding), 
[icuNormalizer](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#icunormalizer), 
[kStemming](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#kstemming), 
[length](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#length), 
[lowercase](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#lowercase), 
[porterStemming](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#porterstemming), 
[regex](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#regex), 
[reverse](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#reverse), 
[shingle](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#shingle), 
[snowballStemming](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#snowballstemming), 
[spanishPluralStemming](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#spanishpluralstemming), 
[stempel](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#stempel), 
[stopword](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#stopword), 
[trim](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#trim), 
[wordDelimiterGraph](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#worddelimitergraph)
