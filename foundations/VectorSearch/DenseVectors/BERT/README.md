## [BERT](http://jalammar.github.io/illustrated-bert/)

*There's about 500 pages that could go into explaining BERT. I'm going to try to abstract 499 of those pages out so you can get the idea behind it*

*this particular readme is also a work in progress*

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

*flow from output logits to token_id will look like such*

![](/images/BERT/MLM_pred.png)

--------------------------------------------------------------------------------------------------------------------------------------------------------

This flow might have left you with quite a few questions.
1.  Why are we masking tokens? - Masked Language Model
    - Masked Language Model 
      - The purpose of the Masked Language Model is to, simply put, teach a model to understand `what is language` and `what is context`. 
      - The user provides BERT an input sequence of tokens and masks ~15% of the words to then allow for BERT to learn and predict the correct output sequence by learning the semantics behind the sequence. 
      -BERT will also need to understsand relationships across sentences as well which can be done using Next Sentence Prediction (see below)

    - Next Sentence Prediction
      - Many important downstream tasks such as Question Answering (QA) and Natural Language Inference (NLI) are based on understanding the relationship between two sentences, which is not directly captured by language modeling
      - Bert can predict sentences given context. BERT can take in two sentences and determine whether the second sentence follows the first or not (binary classification). 
      - This allows for BERT to better understand context across multiple sequences of tokens rather than just one sequence of tokens.
      - This also allows for BERT to do next sentence prediction tasks that can be trivially generated from any monolingual corpus
    
    
2.  What determines the BERT output vector size?
    - The number of hidden units (aka neurons or features) within BERT is 768 and cannot be modified without retraining the model. 
4.  What's the responsibility of the feed forward neural network that produces our logits?
    - The Vector output from BERT can do a lot more than just guessing tokens. It has provided context to the vectors but it hasn't done anything with that information yet. A feedforward neural network can take that 768 sized vector and make decisions upon it such as
        - Guessing whether the sequence is spam or not spam
        - Guessing whether the sequence is factual or not factual
        - Guessing the masked word(s)
        - Sentiment Analysis
        - etc.
    - Feed Forward neural network helps a lot in finding the more contextual information related to particular pairs of words in sequences. It's meant to improve the accuracy of the overall model. [You can learn more about the feed forward network's responsibility in this video here.](https://www.youtube.com/watch?v=YIEe7d7YqaU)

--------------------------------------------------------------------------------------------------------------------------------------------------------

### What makes BERT so powerful?

- We’ve established that the problem with Word2Vec is that it creates context-independent embeddings, meaning there is just one vector representation for each word. BERT generates embeddings that allow us to have multiple vector representations for the same word based on the context it was used, thus making them context-dependent. This allows us to distinguish and capture different semantics of words.
- Additionally, BERT learns representations at a “subword” level, meaning that even though a BERT model will have a finite vocabulary space, it can provide support for many words defined outside its vocabulary.
  - example: `embeddings` => `em`, `##bed`, `##ding`, `##s`.
  - You can use and reuse these tokens to represent words across the corpus. 

Example of context dependence

![](/images/BERT/context.png)

--------------------------------------------------------------------------------------------------------------------------------------------------------
*Additional Resources*
- [Explain BERT to me like I'm 5 video](https://www.youtube.com/watch?v=xI0HHN5XKDo)
- [Explain BERT to me like I'm 5 article](https://medium.com/@samia.khalid/bert-explained-a-complete-guide-with-theory-and-tutorial-3ac9ebc8fa7c)
- [Coding Exercise]() (coming soon)

