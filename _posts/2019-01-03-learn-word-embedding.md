[embedding1]: {{ site.baseurl }} /images/embedding_1.png
[embedding2]: {{ site.baseurl }} /images/embedding_2.png
[embedding3]: {{ site.baseurl }} /images/embedding_3.png
[embedding4]: {{ site.baseurl }} /images/embedding_4.png
[embedding5]: {{ site.baseurl }} /images/embedding_5.png

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


