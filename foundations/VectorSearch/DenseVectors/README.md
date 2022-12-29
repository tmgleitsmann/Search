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

Where values closer to 1 correlate to tigther relationships between the token and the feature and values closer to 0 correlate to the opposite.

-------------------------------------------------------------------------------------------------------------------------------------

Earlier in the Sparse Vector section I listed off a few machine learning models (neural networks) that can provide much more descriptive vectors than the Bag of Words model we covered. These models are more descriptive in that they produce dense vectors as outputs through *feature learning* during the neural network training process. We can learn a little more about those machine learning models below.

| # | Label                                                       | Description |
|:--|:------------------------------------------------------------|:-----------|
| 1 | [Word2Vec](Word2Vec)  | High level overview of the Word2Vec neural network model |
| 2 | [GloVe](GloVe)     | High level overview of the GloVe neural network model|
| 3 | [BERT](BERT)     | High level overview of the BERT neural network model |

