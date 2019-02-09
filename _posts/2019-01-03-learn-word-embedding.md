[embedding1]: {{ site.baseurl }} /images/embedding_1.png
[embedding2]: {{ site.baseurl }} /images/embedding_2.png
[embedding3]: {{ site.baseurl }} /images/embedding_3.png
[embedding4]: {{ site.baseurl }} /images/embedding_4.png
[embedding5]: {{ site.baseurl }} /images/embedding_5.png
[embedding7]: {{ site.baseurl }} /images/embedding_7.png
[embedding8]: {{ site.baseurl }} /images/embedding_8.png
[embedding9]: {{ site.baseurl }} /images/embedding_9.png
[embedding10]: {{ site.baseurl }} /images/embedding_10.png

After learning and using NLP for a while, it’s good to summarise some key concepts in NLP. I will introduce and explain word embedding in this blog. Before getting to this, let’s first look at some concepts:

**Bag of words:**

Treat each document as a set: As one of ways of feature representation, the bad of words model treat each document as an un-ordered collection or bag of words. Here a document is the unit of text that you want to analyse. First, you still have to do all data cleaning first, and then treat the resulting tokens as an un-ordered collection or set. 

The disadvantage of it will be: keeping these as separate sets in very inefficient, and they are of different sizes, and can contains different words, so hard to compare. And also, ignore the fact that some words occur multiple times in a document.

A more useful approach is to turn each document into a vector of numbers representing how many times each word occurs in a document. A set of documents is known as a corpus, and this gives the context for the vectors to be calculated.

![embedding1]
 
So first, collect all the unique words present in your corpus to form vocabulary. Arrange these words in some order, and let them form the vector element positions or columns of a table, and assume each document is a row. Then count the number of occurrences of each word in each document and enter the value in the respective column. At this stage, it’s easier to think of this as a document-term matrix, illustrating the relationship between documents in rows and words or terms in columns. Each element is term frequency meaning how frequently does this term occur in this document.

![embedding2]

**Comparison:**
 
***Dot product:***
a common way is to check the dot product which is the sum of products of corresponding elements. Greater the dot product, more similar the two vectors are. One downside of dot product is it only captures the portions of overlap. It’s not affected by other values that are not uncommon. So pairs that are very different can end up with the same product as ones that are identical.  
***Cosine similarity:***
we divide the dot product of two vectors by the product of their magnitudes or Euclidean norms. If you think of these vectors as arrows in some n-dimensional space, then this is equal to the cosine of the angle theta between them. Identical vectors have cosine equal to one, orthogonal vectors have cosine equal zero. For vectors that are exactly opposite, it’s minus one.

![embedding3]

**TF-IDF:**

One disadvantage of bag of words is to treat every word as being equally important. Therefore, we can compensate by counting the number of documents in which each word occurs, this can be called document frequency, and then dividing term frequencies by the document frequency of that term. And this gives us a metric that is proportional to the frequency of occurrence of a term in a document, but inversely proportional to the number of document it appears in. It highlights the words that are more unique to a document, and thus better for characterising it. 
e.g.

![embedding4]

**TF-IDF transform:**
![embedding5]

The most commonly used form of TF-IDF defines term frequency as the raw count of a term t, in a document d, divided by the toal number of terms in d. And inverse document frequency as logarithm of the total number of documents in the collection d, divided by the number of documents where t is present. Several variations exist that try to normalise, or smooth the resulting values, or prevent edge cases such as divide-by-zero errors. Overall, TF-IDF is an innovative approach to assigning weights to words that signify their relevance in documents.  

**One-hot encoding:**
Not very practical given a large vocabulary of corpus, because the size of word representation grows with the number of words. 

**Word Embeddings:**

What we need as a way to control the size of our word representation by limiting it to a fixed-size vector. In other words, we want to find an embedding for each word in some vector space and we wanted to exhibit some desired properties. For example, if two words are similar in meaning, they should be closer to each other compared to words that are not. 

And if two pairs of words have a similar difference in their meanings, they should be approximately equally separated in the embedded space. We could use such a representation for a variety of purposes like finding synonyms and analogies, identifying concepts around which words are clustered, classifying words as positive, negative, neutral, etc.

**Word2Vec:**

Word2Vec is perhaps the most popular examples of word embeddings used in practice. It transforms words to vectors. The core idea behind this is, a model that is able to predict a given word, given neighbouring words, or vice versa, predict neighbouring words for a give word is likely to capture the contextual meanings of words very well. 

Two flavours of Word2Vec models, one where you are given neighbouring words called continuous bag of words, and the other where you are given the middle word called Skip-gram. 

![embedding7]

In skip-gram model, you pick any word from a sentence, convert it into a one-hot encoded vector and feed it into a neural network or some other probabilistic model that is designed to predict a few surrounding words, its context, using a suitable loss function, optimise the weights or parameters of the model and repeat this till it learns to predict context words as best as it can. Now, take the intermediate representation like a hidden layer in a neural network, the outputs of that layer for a given word become the corresponding word vector. The continuous bag of words variation also uses a similar strategy. 

![embedding8]

This yields a very robust representation of words, because the meaning of each word is distributed throughout the vector. The size of vector is up to you, how you want to tune performance versus complexity. It remains constant no matter how many words you train on, unlike bag of words, for instance, where the size grows with the number of unique words. And once you pre-train a large set of word vectors, you can use them efficiently without having to transform again and again, just store them in a lookup table. Finally, its ready to be used in deep learning architectures!! For example, it can be used as the input vector for recurrent neural nets. 

**GloVe:**

Word2Vec is just one type of forward embedding, and there are other related approaches that are also promising. GloVe or global vectors for word representation is such an approach that tries to directly optimise the vector representation of each word just using co-occurrence statistics, unlike word2vec which sets up an ancillary prediction task. 

First, the probability that word ji appears in the context of word i is computed, pj given i for all word pairs ij in a given corpus. What do we mean by j appears in context of i?

Simply that word j is present in the vicinity of word i, or a few words away. We count all such occurrences of i and j in our text collection, and then normalise account to get a probability. Then, a random vector is initialised for each word, actually two vectors, one for the word when it’s acting as a context, and one when it’s acting as the target. Now for any pair of words, ij, we want the dot product of their word vectors, we want the dot product of their word vectors, w_i times w_j, to be equal to their co-occurrence probability. Using this as our goal and a suitable last function, we can iteratively optimise these word vectors. The result should be a set of vectors that capture the similarities and differences between individual words. 

![embedding9]

If you look at it from another point of view, we are essentially factorising the co-occurrence probability matrix into two smaller matrices. This is the basic idea of GloVe.

Why co-occurrence probabilities?:

![embedding10]

For example, we would come across solid more often in the context of ice than steam. But water would occur in either context with roughly equal probability. Given a large corpus, you’ll find that the ratio of P solid given ice to P solid given steam is much greater than 1, while the ratio of P water given ice and P water given steam is close to 1. Thus we see that co-occurrence probabilities already exhibit some of the properties we want to capture. In fact, one refinement over using raw probalitiy values is to optimise for the ratio of probabilities. Now there are lots of subtleties here, not the least of which is the fact that the co-occurrence probability matrix is huge. At the same time, co-occurrence probability values are typically very low, so it makes sense to work with the log of these values. 



