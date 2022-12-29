## [BERT](http://jalammar.github.io/illustrated-bert/)

Bert (Bidirectional Encoder Representations from Transformers) is a [machine learning model](https://arxiv.org/pdf/1810.04805v2.pdf) that allows you to generate *contextualized* embeddings, solving the issues from Word2Vec and GloVe. Today, some of the world's most powerful search services are powered by BERT, including Google Search. 

The architecture for BERT is quite complex at it is trained against two models simultaneously. 
- Masked Language Model 
  - The user provides BERT an input sequence of tokens and masks ~15% of the words to then allow for BERT to learn and predict the correct output sequence. 
  - During the training process BERT will need to learn the semantics behind a sequence of words. 
  - BERT will output logits that map to the corpus we are training against, then apply a softmax activation to those logits to get a probability distribution of which logit is most likely to be plugged into a masked word, and then the argmax will provide the logit with the highest probability which we can then use to map back to the token it represents in our corpus. 
  
  ![](/images/BERT/MLM_pred.png)
