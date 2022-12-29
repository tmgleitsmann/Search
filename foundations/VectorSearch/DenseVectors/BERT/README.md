## [BERT](http://jalammar.github.io/illustrated-bert/)

Bert (Bidirectional Encoder Representations from Transformers) is a [machine learning model](https://arxiv.org/pdf/1810.04805v2.pdf) that allows you to generate *contextualized* embeddings, solving the issues from Word2Vec and GloVe. Today, some of the world's most powerful search services are powered by BERT, including Google Search. 

The architecture for BERT is quite complex at it is trained against two models simultaneously
1. The user provides a tokenizer with a sequence of characters. 
2. The tokenizer produces tokens out of the sequence and then passes these tokens along to a masking function
3. The masking function will mask roughly 15% of the tokens before passing the tokens to the BERT model. 
4. BERT will output a set of vectors of length 768 (usually, depending on the BERT model).
5. These vectors get passed into a feed forward neural network that outputs logits of size of our corpus size (generally 30,500)
6. Each logit will then pass through a softmax function to get a probability distribution and then an argmax function to get the token id
7. Decode the token id using our tokenizer to produce the word in english 

![](/images/BERT/MLM.png)

- Masked Language Model 
  - The user provides BERT an input sequence of tokens and masks ~15% of the words to then allow for BERT to learn and predict the correct output sequence. 
  - During the training process BERT will need to learn the semantics behind a sequence of words. 
  - BERT will output logits that map to the corpus we are training against, then apply a softmax activation to those logits to get a probability distribution of which logit is most likely to be plugged into a masked word, and then the argmax will provide the logit with the highest probability which we can then use to map back to the token it represents in our corpus. 
     ![](/images/BERT/MLM_pred.png)
     

