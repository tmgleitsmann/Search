# What are Dense Encodings (Vectors)?

***If you haven't reviewed the Sparse Vectors section of the repository, please do so first***

Sparse vectors can be incredibly powerful but we just raised the issue they have with space and time complexity. This was due to the fact that a majority of the vector itself doesn’t hold any useful information, thus making it sparse. Another problem it has is that it is an unrealistic and impractical way of capturing true semantics for words. It’s “all or nothing” way of representing data (1s and 0s) would prove problematic for capturing features that lie in-between.

For example, let’s say we had a algorithm that gauged for features like 
- Cold
- Wet
- Fast
- Expensive

A more accurate way of representing vectors as an output of this algorithm that is even more descriptive than sparse vectors might be

| *token*   | cold | wet | fast | expensive |
|:----------|:-----|:----|:-----|:----------|
| Ferrari   | .3   | .1  | .8   | .9        |
| Shark     | .8   | .99 | .3   | .2        |
| Lightning | .01  | .01 | .99  | .01       |

Earlier in the Sparse Vector section I listed off a few machine learning models (neural networks) that can provide much more descriptive vectors than the Bag of Words model we covered. These models are more descriptive in that they produce dense vectors as outputs through *feature learning* during the neural network training process. We can learn a little more about those machine learning models below.

| # | Label                                                       | Description |
|:--|:------------------------------------------------------------|:-----------|
| 1 | [Word2Vec](foundations/VectorSearch/Word2Vec)  | High level overview of the Word2Vec neural network model |
| 2 | [GloVe](foundations/VectorSearch/GloVe)     | High level overview of the GloVe neural network model|
| 3 | [BERT](foundations/VectorSearch/BERT)     | High level overview of the BERT neural network model |

-------------------------------------------------------------------------------------------------------------------------------------

## Word2Vec

Computers cannot understand the meaning behind text directly, so text needs to be represented by a set of numbers. A set of numbers is a vector. Each number within a vector represents a feature, so TYPICALLY the more dense a vector, the more accurate the meaning tying it back to its original text. You can think of vectors as capturing semantics.

Think to yourself: `King - Man + Woman = Queen`

This equation would make sense to us if we take all the characteristics of a King, subtract the characteristics of a Man and add the characteristics of a Woman. This would leave us with *something* that reflects the characteristics of a Queen. 

Computers can understand this equation if we convert the text to numbers, or vectors rather. This is what we've been covering since the beginning of the Sparse Vectors section of the repository; the idea behind capturing the semantics behind text. 

Word2Vec introduces a different way of capturing features of tokens through a different type of algorithm. It is a particular machine learning model that can accept text from a corpus and produce these word embeddings (vectors). This is super powerful because it allows us to represent text on an n-dimensional plane where n represents the features within our vectors then we model text relationships. This allows us to then do things like compare or predict text, mathematically manipulate vectors, etc.

By being able to maniuplate these incredibly sophisticated vectors, we can build out services like
 - Reccomendation Engines: Based off of the searched vector, find the vectors most similar to then recommend back to the user. 
 - Semantic Search: Search for even more relvenat search by understanding the context behind what the user may be inputting.
 
 This can be done by manipulating vectors in a multitude of ways
 1. Taking the Euclidean Distance between ends of vectors to see how similar/dissimilar they are
 2. Calculating the Cosine Similarity (angle) vetween vectors to see how similar/dissimilar they are
 3. Efficiently measure similarity based on both angle and magnitude
 
 ### Word2Vec comes in two different architecture (models)
 
 ![](/images/Word2Vec/models.png)
 
- Continuous Bag of Words: A model architecture that predicts the `focus word` given `context` as input.
  - The idea is to determine the context as vectors and them sum them to determine the focus word
![](/images/Word2Vec/CBOW.png)
- Skip Gram (also reffered to as Skip N-Gram): A model architecture that predicts the  `context` given a `focus word`
  - This is the inverse of Continuous Bag of Words. You take a focus word to try and predict context words one-by-one. 
![](/images/Word2Vec/SkipGram.png)

-------------------------------------------------------------------------------------------------------------------------------------

### [How does Word2Vec Work?](https://www.youtube.com/watch?v=QyrUentbkvw)

*I highly encourage watching this video, regardless of if you are grasping the concepts behind Word2Vec or not*

We’ve established the ideas of focus words and context words. The semantics of these matrices will need to be learned by a machine learning model. So what needs to be created is an algorithm that predicts the probability of a focus word given context words (for CBOW) or predicting context words given a focus word (for Skip Gram).

We won’t go through the algorithms that actually need to be learned here, but these can be modeled as logistical regression equations. This allows us to build a machine learning model that can learn to predict a focus word given context words or context words given a focus word. 

> **NOTE:** When learning, we don’t take the entire context, whether that be a sentence, paragraph, or document. We take a context window that we can define to be a fixed length. This will be a moving window. Typically the window will denote the number of words before and after your target word, but there are scenarios in which the window denotes every word, including the target word like below. 
- example: There lived a king called Ashoka in India.
 - window = 3 : lived, a → there | [There <-- focus word] [lived a <-- context words] king called Ashoka in India
 - window = 3 : a, king → lived | There [lived <-- a king] [lived a <-- context words] called Ashoka in India
 - etc.
 
 -------------------------------------------------------------------------------------------------------------------------------------
 
### Word2Vec Exercie
 [Exercise](): Lets train our own Word2Vec model against the Amazon Product Reviews dataset and then measure the similarity between like vectors
 
 -------------------------------------------------------------------------------------------------------------------------------------

### The Major Drawback of Word2Vec

- Only generates fixed embedding vector: Problematic for words that have more than one meaning. Word2Vec struggles to generate contextualized meaning of a word

  > Ie. bank ⇒ bank for money, bank for basketball shot, bank for area by a river.
