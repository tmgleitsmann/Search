# What are Sparse Encodings (Vectors)?

What we’ve learned up to this point about Full Text Search and Lucene only scratches the surface of what relevance search can and should accomplish. So far we’ve established a sophisticated indexing and scoring model that can be used to bring powerful search capabilities to an end user. But what if we need to provide our user with more than just text search? What if we need to service not just the text, but the semantics behind the text as well?

Computers cannot comprehend semantics behind pure text. The text needs to be converted into numbers first. If you think about it, Lucene already does this with its scoring model. We assign relevance as a number to its associated matching text with TF-IDF scoring. What we’d be looking to do by adding semantic understanding to our full text search, is just replacing how we associate relevance. We’d be replacing TF-IDF with another, more robust algorithm altogether. 

Recently these algorithms have developed into machine learning models. Throughout this repository you’ll learn the basics behind what these machine learning models look to accomplish and how, but anytime you see one you can think of it as a multivariate equation. It receives input, that input activates neurons in a neural network which generates an output. 

The output of these models is a vector, hence why the title of this ReadME is titled Sparse Vectors. These vectors are simply numeric representations of the meaning (semantics) behind the text the model received as input. For example if I fed the model sequences of text
1. I like to play football
2. Did you go outside to play tennis
3. John and I play tennis

I can expect output such as…
| Sentence #| Play | Tennis | To | I | Football | Did | You | Go |
|:----------|:-----|:-------|:---|:--|:---------|:----|:----|:---|
| 1         | 1    | 0      | 1  | 1 | 1        | 0   |0    | 0  |
| 2         | 1    | 1      | 1  | 0 | 0        | 1   |1    | 1  |
| 3         | 1    | 1      | 0  | 1 | 0        | 0   |0    | 0  |

This isn’t a super sophisticated output. What we have is the words in our corpus that exist within a particular text sequence. For example, the input for Sentence 1 would generate the output [1, 0, 1, 1, 1, 0, 0, 0], where the 1s denote whether a particular word within a corpus exists in the input and the 0s denote absences of particular words. This all or nothing numeric representation of features is what we call “Sparse” Vectors, since a small portion of a vector actually provides useful information (thus sparse). 

> **NOTE**: This vector representation is known as the “Bag of Words” model and does not require a machine learning model to construct. However there are machine learning models (see list below) that can provide much more elaborate vectors. We will tackle those shortly.
> - Word2Vec
> - GloVe
> - BERT
> - and more...

The sparse vector denoted from the table above can help us understand semantic relationships between text given the sentences passed as input. For example, from looking at this table I might be able to infer that the token `Play` can be closely associated with the tokens `Tennis` and/or `Football` since they appear in the same sentences frequently. 

-------------------------------------------------------------------------------------------------------------------------------------

## Sparse Vector Exercise

Check out this [exercise](https://github.com/esteininger/vector-search/blob/master/foundations/sparse-vector-tutorial/Sparse%20Vector%20Tutorial.ipynb) Ethan Steininger put together for constructing your own Sparse Vectors, effectively building a feature extraction engine (using the bag-of-words approach) that stores tokens as 1's and 0's to then become searchable.

### The Drawbacks to Sparse Vectors
- Very large vectors require a lot of memory, and some very large vectors that we wish to work with are sparse. Most of the information, since it’s 0s, is useless.
- Performing any sort of computations on these vectors is wasteful because most of the arithmetic operations devoted to solving a set of equations or inverting the vectors involve zero operands.
