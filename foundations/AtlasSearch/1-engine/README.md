# Build a FTS Engine in Python

Review the [python notebook](engine.ipynb) to follow along with all of the code. 

## Review the Components
FTS engines will require several different pieces. At minimum we'll require...
1. Analyzer: Composed of character filter(s), tokenizer(s), and token filter(s)
2. An Inverted Index to map our tokens back to their original documents
3. A Relevance Scorer

Once this functionality is defined, we can build a search function that utilizes them all to find the most relevant documents sorted by relevance score.

## Analyzer

### The separate functions that our analyzer will call are defined below

``` python
def lowercase_filter(tokens):
  return [token.lower() for token in tokens]

def punctuation_filter(tokens):
  PUNCTUATION = re.compile('[%s]' % re.escape(string.punctuation))
  return [PUNCTUATION.sub('', token) for token in tokens]

def stem_filter(tokens):
  STEMMER = Stemmer.Stemmer('english')
  return STEMMER.stemWords(tokens)
    
def tokenize(text):
  return text.split()

def stopword_filter(tokens):
  STOPWORDS = set(['the', 'be', 'to', 'of', 'and', 'a', 'in', 'that', 'have',
                     'I', 'it', 'for', 'not', 'on', 'with', 'he', 'as', 'you',
                     'do', 'at', 'this', 'but', 'his', 'by', 'from', 'wikipedia'])
  return [token for token in tokens if token not in STOPWORDS]
```

### The Analyzer itself will call on the functions above

``` python
def analyze(text):
  tokens = tokenize(text)
  tokens = lowercase_filter(tokens)
  tokens = punctuation_filter(tokens)
  tokens = stopword_filter(tokens)
  tokens = stem_filter(tokens)

  return [token for token in tokens if token]
```


## Index

``` python
def index():
  index={}
  for document in documents:
    for token in analyze(document['title']):
      if token not in index:
        index[token] = set()
      index[token].add(document['_id']['$oid'])
  return index
```

## Scorer

Since we're scoring with TF-IDF, we'll need functions to determine the term frequency and inverse document frequency. 

``` python
def term_frequency(document, token):
  return analyze(document['title']).count(token)

def inverse_document_frequency(documents, token):
  math.log10(len(documents) / len(index().get(token)))
  
```

## Source
https://bart.degoe.de/building-a-full-text-search-engine-150-lines-of-code/

