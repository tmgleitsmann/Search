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


------------------------------------------------------------------------------------------------------------------------------------------------------------------


For a majority of the indexes we've created, we've used the visual index editor. We can alternatively choose to use the JSON editor and define our analyzers in the following format

```json
"analyzers": [
  {
    "name": "<name>",
    "charFilters": [ "<list-of-character-filters>" ],
    "tokenizer": {
      "type": "<tokenizer-type>"
    },
    "tokenFilters": [ "<list-of-token-filters>" ]
  }
]
```

## Exercise

Let’s insert a few documents in the `custom.minutes` namespace to play with. Connect to your cluster from the shell and insert the following four documents. 

```shell
use custom
db.minutes.insertMany(
[
  {
    "_id": 1,
    "page_updated_by": {
      "last_name": "AUERBACH",
      "first_name": "Siân",
      "email": "auerbach@example.com",
      "phone": "(123)-456-7890"
    },
    "title": "The team's weekly meeting",
    "message": "try to siGn-In",
    "text": {
      "en_US": "<head> This page deals with department meetings. </head>"
    }
  },
  {
    "_id": 2,
    "page_updated_by": {
      "last_name": "OHRBACH",
      "first_name": "Noël",
      "email": "ohrbach@example.com",
      "phone": "(123) 456 0987"
    },
    "title": "The check-in with sales team",
    "message": "do not forget to SIGN-IN",
    "text" : {
      "en_US": "The head of the sales department spoke first.",
      "fa_IR": "ابتدا رئیس بخش فروش صحبت کرد"
    }
  },
  {
    "_id": 3,
    "page_updated_by": {
      "last_name": "LEWINSKY",
      "first_name": "Brièle",
      "email": "lewinsky@example.com",
      "phone": "(123).456.9870"
    },
    "title": "The regular board meeting",
    "message": "try to sign-in",
    "text" : {
      "en_US": "<body>We'll head out to the conference room by noon.</body>"
    }
  },
      {
    "_id": 4,
    "page_updated_by": {
      "last_name": "LEVINSKI",
      "first_name": "François",
      "email": "levinski@example.com",
      "phone": "123-456-8907"
    },
    "title": "The daily huddle on StandUpApp2",
    "message": "write down your signature or phone №",
    "text" : {
      "en_US": "<body>This page has been updated with the items on the agenda.</body>" ,
      "es_MX": "La página ha sido actualizada con los puntos de la agenda.",
      "pl_PL": "Strona została zaktualizowana o punkty porządku obrad."
    }
  }
])
```

Notice how the phone numbers are not standardized. They have "-", "(", ")", ".", and " " in and around the numbers. 
If we wanted to create an index to support the search of a phone number, we might want to introduce a mapping character filter followed by a keyword tokenizer on the `phone` field. 

### 1. First let's setup our Lucene field mappings 
(This is NOT character mapping for the character filter). We're just specifying where in the docoument we want to index against, with what analyzer, and the data type that `phone` is. 

```json
  "mappings": {
    "fields": {
      "page_updated_by": {
        "fields": {
          "phone": {
            "analyzer": "mappingAnalyzer",
            "type": "string"
          }
        },
        "type": "document"
      }
    }
  }
```

### 2. Next, let's setup the analyzer portion of the index.
We will first start off with defining the character filter - *mapping*
Then, we will define the tokenizer. Since we will want to query against the phone number as if it were a keyword, we will use the `keyword` tokenizer.
After tokenizing, is there anything else that we'll need to filter for? Once we have the *pure* number, we can index it as is. There is no need for a token filter. 

- name the analyzer, `mappingAnalyzer`
- map the following characters blank
  - "-", "(", ")", ".", " "
- set the tokenizer to `keyword`

```json
{
  "mappings": {
    "fields": {
      "page_updated_by": {
        "fields": {
          "phone": {
            "analyzer": "mappingAnalyzer",
            "type": "string"
          }
        },
        "type": "document"
      }
    }
  },
  "analyzers": [
    {
      "name": "mappingAnalyzer",
      "charFilters": [
        {
          "mappings": {
            "-": "",
            ".": "",
            "(": "",
            ")": "",
            " ": ""
          },
          "type": "mapping"
        }
      ],
      "tokenizer": {
        "type": "keyword"
      }
    }
  ]
}
```

### 3. Now let's Create and Test the Index

Create the following index using the JSON Editor in the Search tab of the MongoDB Atlas UI. The namespace it should be created on is `custom.minutes`.

<img src="/images/AtlasSearch/16-custom-analyzers/customIndex.png" style="height: 80%; width:80%;"/>

Next, navigate to the `collections` tab and create an aggregation in the `custom.minutes` namespace. 
  1. Create a $search stage to match against the number `1234567890` on the `page_updated_by.phone` field.
  2. Project only the number for visibility. 
  
  ```json
  [{
   "$search": {
    "index": "default",
    "text": {
     "query": "1234567890",
     "path": "page_updated_by.phone"
    }
   }
  }, {
   "$project": {
    "phone": "$page_updated_by.phone"
   }
  }]
  ```
  
  <img src="/images/AtlasSearch/16-custom-analyzers/aggregation.png" style="height: 50%; width:50%;"/>

  The result should be pretty straightforward. We are returning the original contents of the documents we matched against. 
  
  <img src="/images/AtlasSearch/16-custom-analyzers/result.png" style="height: 50%; width:50%;"/>


