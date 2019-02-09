[embedding1]: {{ site.baseurl }} /images/embedding_1.png
[embedding2]: {{ site.baseurl }} /images/embedding_2.png
[embedding3]: {{ site.baseurl }} /images/embedding_3.png
[embedding4]: {{ site.baseurl }} /images/embedding_4.png
[embedding5]: {{ site.baseurl }} /images/embedding_5.png

After learning and using NLP for a while, it’s good to summarise some key concepts in NLP. I will introduce and explain word embedding in this blog. Before getting to this, let’s first look at some concepts:

Bag of words:

Treat each document as a set: As one of ways of feature representation, the bad of words model treat each document as an un-ordered collection or bag of words. Here a document is the unit of text that you want to analyse. First, you still have to do all data cleaning first, and then treat the resulting tokens as an un-ordered collection or set. 

The disadvantage of it will be: keeping these as separate sets in very inefficient, and they are of different sizes, and can contains different words, so hard to compare. And also, ignore the fact that some words occur multiple times in a document.

A more useful approach is to turn each document into a vector of numbers representing how many times each word occurs in a document. A set of documents is known as a corpus, and this gives the context for the vectors to be calculated.

![embedding1]
 
