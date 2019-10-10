
+++
title = "Implementing search engine using TF-IDF"
summary = "Describes How I implemented a simple search engine using TF-IDF to search Lifestyle products"
+++
 
Searching for something is an inevitable part now. So, let's see, the basics of implementing a search engine. Here we will search for Life Style products which will based on [this](https://www.kaggle.com/paramaggarwal/fashion-product-images-small) data set which contains information on thousands of life style products. First, we will see _How to create **Index** and **rank** the results we get from the search_

But before that let's talk about the pre-processing of our data.

## Data Pre-processing
The data set I choose had a csv file with products in the row. It had some small attributes in each column. To search over the properties and product description I combined all the properties and the description in one column. There were some rows with empty description filed. I removed those products.


Next, I removed the most common words which itself doesn't carry any significance like 'a', 'an', 'to', 'for'. These are called stopwords. Next we will use [Lemmatization](https://nlp.stanford.edu/IR-book/html/htmledition/stemming-and-lemmatization-1.html) (Finding the root word of a term) and [Stemming](https://nlp.stanford.edu/IR-book/html/htmledition/stemming-and-lemmatization-1.html) (Trimming the last part of a word based on set of rule) to find similar words.

Following code applies these techniques:

```
import nltk
nltk.download('stopwords')
nltk.download('wordnet')

from nltk.corpus import stopwords
from nltk.stem import PorterStemmer
from nltk.stem import WordNetLemmatizer

def prepareParams(self):
  self.stopwords = set(stopwords.words('english'))
  self.dataFile = STYLE_WITH_DESC_N_TITLE
  self.indexFile = INVERTED_IDX_FILE
  self.stemmer = PorterStemmer()
  self.lemmatizer = WordNetLemmatizer()
  
def getTerms(self, doc):
  doc = doc.lower()
  doc = re.sub(r'[^a-z0-9 ]',' ',doc) #put spaces instead of non-alphanumeric characters
  terms = doc.split()

  terms = [term for term in terms if term not in self.stopwords]
  terms = [self.lemmatizer.lemmatize(term) for term in terms]
  terms = [self.stemmer.stem(term) for term in terms]
  return terms

```

## Creating the index
We will now create the index of our dataset using Inverted Index, TF(Term frequency) and IDF(Inverse Document Frequency)

-----------------------------------------------
### Inverted Index
An inverted index is a data structure that we build while parsing the documents that we are going to answer the search queries on. Given a query, we use the index to return the list of documents relevant for this query. The inverted index contains mappings from terms (words) to the documents that those terms appear in. Each vocabulary term is a key in the index whose value is its postings list. A term’s postings list is the list of documents that the term appears in.


### TF-IDF
Tf-idf is a weighting scheme that assigns each term in a document a weight based on its term frequency (tf) and inverse document frequency (idf).  The terms with higher weight scores are considered to be more important. It’s one of the most popular weighting schemes in Information Retrieval.


### Term Frequency – tf
Term Frequency is calculated for a term t in document d. It is basically the number of occurrences of the term in the document.

$$ {tf_t}_d = {N_t}_d $$

We can see that as a term appears more in the document it becomes more important, which is logical. However, there is a drawback, by using term frequencies we lose positional information. The ordering of terms doesn’t matter, instead the number of occurrences becomes important. This is known as the bag of words model, and it is widely used in document classification. In bag of words model, the document is represented as an unordered collection of words. However, it doesn’t turn to be a big loss. Of course we lose the semantic difference between “Bob likes Alice” and “Alice likes Bob”, but we still get the general idea.

We can use a vector to represent the document in bag of words model, since the ordering of terms is not important. There is an entry for each unique term in the document with the value being its term frequency. For the sake of an example, consider the document “computer study computer science”. The vector representation of this document will be of size 3 with values [2, 1, 1] corresponding to computer, study, and science respectively. We can indeed represent every document in the corpus as a k-dimensonal vector, where k is the number of unique terms in that document. Each dimension corresponds to a separate term in the document. Now every document lies in a common vector space. The dimensionality of the vector space is the total number of unique terms in the corpus. We will further analyze this model in the following sections. The representation of documents as vectors in a common vector space is known as the vector space model and it’s very fundamental to information retrieval. It was introduced by Gerard Salton, a pioneer of information retrieval. Google’s core ranking team is led by Amit Singhal, who was the PhD student of Salton at Cornell University.

While using term frequencies if we use pure occurrence counts, longer documents will be favored more. Consider two documents with exactly the same content but one being twice longer by concatenating with itself.  The tf weights of each word in the longer document will be twice the shorter one, although they essentially have the same content. To remedy this effect, we length normalize term frequencies. So, the term frequency of a term t in document D now becomes:

$$ {tf_t}_d = \dfrac{{N_t}_d}}{||D||}$$

||D|| is known as the Euclidean norm and is calculated by taking the square of each value in the document vector, summing them up, and taking the square root of the sum. After normalizing the document vector, the entries are the final term frequencies of the corresponding terms. The document vector is also a unit vector, having a length of 1 in the vector space.

Inverse Document Frequency – idf
We can’t only use term frequencies to calculate the weight of a term in the document, because tf considers all terms equally important. However, some terms occur more rarely and they are more discriminative than others. Suppose we search for articles about computer vision. Here the term vision gives us more information about the intent of the query, instead of the term computer. We don’t simply want articles that are about computers, we want them to be about vision. If we purely use tf values then the term computer will dominate because it’s a more common term than vision, and the articles containing computer will be ranked higher. To mitigate this effect, we use inverse document frequency. Let’s first see what document frequency is. The document frequency of a term t is the number of documents containing the term:

df_t = N_t  

Note that the occurrence counts of the term in the individual documents is not important. We are only interested in whether the term is present in a document or not, without taking into consideration the counts. It’s like a binary 0/1 counting. If we were to consider the number of occurrences in the documents, then it’s called collection frequency. But document frequency proves to be more accurate. Also note that term frequency is a document-wise statistic while document frequency is collection-wise. Term frequency is the occurrence count of a term in one particular document only; while document frequency is the number of different documents the term appears in, so it depends on the whole corpus. Now let’s look at the definition of inverse document frequency. The idf of a term is the number of documents in the corpus divided by the document frequency of a term. Let’s say we have N documents in the corpus, then the inverse document frequency of term t is:

idf_t = \dfrac{N}{df_t} = \dfrac{N}{N_t}  

This is a very useful statistic, but it also requires a slight modification. Consider a corpus with 1000 documents. A term appears in 10 documents and another term appears in 100, so the document frequencies are 10 and 100 respectively. The inverse document frequencies are 100 and 10. Idf is 100 for the term that has a df of 10 (1000/10), and idf is 10 for the document with df 100 (1000/100) by definition. Now as we can see the term that appears in 10 times more documents is considered to be 10 times less important. It’s expected that the more frequent term to be considered less important, but the factor 10 seems too harsh. Therefore, we take the logarithm of the inverse document frequencies. Let’s say the base of log is 2, than term that appears 10 times less often is considered to be around 3 times more important. So, the idf of a term t becomes:

idf_t = log\dfrac{N}{df_t}  

This is better, and since log is a monotonically increasing function we can safely use it. Notice that idf never becomes negative because the denominator (df of a term) is always less than or equal to the size of the corpus (N). When a term appears in all documents, its df = N, then its idf becomes log(1) = 0. Which is ok because if a term appears in all documents, it doesn’t help us to distinguish between them. It’s basically a stopword, such as “the”, “a”, “an” etc. Also notice the resemblance between idf and the definition of entropy in information theory. In our case p(x) is df/N, which is the probability of seeing a term in a randomly chosen document. And idf is –logp(x). The important result to note is, as more rare events occur, the information gain increases. Which means less frequent terms gives us more information.

Tf-idf scoring
We have defined both tf and idf, and now we can combine these to produce the ultimate score of a term t in document d. We will again represent the document as a vector, with each entry being the tf-idf weight of the corresponding term in the document. The tf-idf weight of a term t in document d is simply the multiplication of its tf by its idf:

tf\mbox{-}idf_{t,d} = tf_{t,d} \cdot idf_t  

Let’s say we have a corpus containing K unique terms, and a document containing k unique terms. Using the vector space model, our document becomes a k-dimensional vector in a K-dimensional vector space. Generally k will be much less than K, because all terms in the corpus won’t appear in a single document. The values in the vector corresponding to the k terms that appear in the document will be their respective tf-idf weights, computed by the formula above. The entries corresponding to the K-k terms that don’t appear in the current document will be 0. Because their tf weight in the current document will be 0, since they don’t occur. Note that their idf scores won’t be 0, because idf is a collection-wise statistic, which depends on all the documents in the corpus. But tf is a document-wise statistic, which only depends on the current document. So, if a term doesn’t appear in the current document, it gets a tf score of 0. Multiplying tf and idf, the tf-idf weights of the missing K-k terms become 0. So, in the end we have a sparse vector with most of the entries being 0. To sum everything up, we represent documents as vectors in the vector space. A document vector has an entry for every term, with the value being its tf-idf score in the document.

We will also represent the query as a vector in the same K-dimensional vector space. It will have much fewer dimensions though, since queries are generally much shorter than the documents. Now let’s see how to find relevant documents to a query. Since both the query and the documents are represented as vectors in a common vector space, we can take advantage of this. We will compute the similarity score between the query vector and all the document vectors, and select the ones with top similarity values as relevant documents to the query. Before computing the similarity scores between vectors, we will perform one final operation as we did before, normalization. We will normalize both the query vector and all the document vectors, obtaining unit vectors.

Now that we have everything we need, we can finally compute the similarity scores between the query and document vectors, and rank the documents. The similarity score between two vectors in a vector space is the the angle between them. If two documents are similar they will be close to each other in the vector space, having a small angle in between. So given the vector representation of the documents, how do we compute the angle between them? We can do it very easily if the vectors are already normalized, which is true in our case, and this technique is called cosine similarity. We take the dot product of the vectors and the result is the cosine value of the angle between them. Remember that when the angle is smaller its cosine value is larger, so when two vectors are similar their cosine similarity value will be larger. This gives us a great similarity metric with higher values meaning more similar and lower values meaning less. Therefore, if we compute the cosine similarity between the query vector and all the document vectors, sort them in descending order, and select the documents with top similarity, we will obtain an ordered list of relevant documents to this query. Voila! We now have a systematic methodology to get an ordered list of results to a query, ranking.








-----------------------------------------------

The reviews contain some html codes that are left as is and need to be converted to utf-8 encoded string with **htlm** library's 
unescape function. Once this is done now its time for text mining, we convert each review string into a list of words with 
the _**split**_ function for a string. We ensure that all words are in **lower case** and must be of minimum length 3. 

The **nltk** library is one of the best and easiest for natural language processing with python which includes **_stopwords.stop("english")_** which can be downloaded and the function is available in the **corpus** function on the module. We use this to remove stop words from our  reviews. The library also consists of **_WordNetLemmatizer()_** using which we create a lemmatizer object and use the **__lemmatize()__** function on each word to find it's root word. This ends the preprocessing step.

**<h2>Creating Vocabulary:</h2>**
<body>We use the inverted index technique which is popular in search engines nowadays. It is very fast as we only calculate similarity for top few reviews.
This is a crucial step, here we traverse through all the preprocessed reviews in our dataset to find unique words in the entire pool. For this we
use a python __dictionary__ as the use hash indexing which is fast. The **keys** of this dictionary are the words. The **value** consists of a _list_, whose first element is the **document frequency** of that word _(number of reviews the word occurs in)_. The second element of this _list_ is another _dictionay_ where **keys** are **ID's** of reviews in which a word occured, **values** are a _list_ with the index position(s)
of the word within the review.
Using this we can now calculate the weights of each word in every review to create posting list for all words. For this we use the **tf-idf** [term frequency-Inverted document frequency](https://medium.freecodecamp.org/how-to-process-textual-data-using-tf-idf-in-python-cd2bbc0a94a3)
The third element in the _list_ is the posting list for every word.
  **Prototype:**
  <pre>
  { word1 : [ docfreq, { docid1:[pos1, pos2, .....], docid2:[pos1, pos2, ....], ....... }, 
            **{ doc1:w1, doc2:w2, .... }** ]
   .
   . 
   .
  }
  </pre>
  Notice we use a position list to store every index of every occurence of a word in a document.
  We get **32742** unique words as features for our dataset.
  <pre>
  
  </pre>
</body> 

<h3> Calculating the tf-idf weights for each word </h3>
<body>
  1. After construction of the first stage of the vocabulary we now calculate the weights of each word.
  2. Term frequency of a word is given with respect to each review it appears in:
  <pre>
  word1-tf-review1 = number of times word1 appears in review1 / number of words in review1
  </pre>
  3. Inverse document frequency for a word is defined by:
  <pre>
  word1-idf = total number of reviews in the dataset / number of reviews word1 occurs in
  </pre>
  4. The weight of a word in a reviewis given by:
  <pre>
  word1-weight-review1 = (1+ log(word1-tf-review1)) / (word1-idf)
  </pre>
  
</body>

**<h2>Query processing and calculating similarity</h2>**

<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
Finally we use the input box we created in our application's search page to retrive user search query and make it undergo the same 
pre-processing as our reviews, converting to lower case, stop word removal and lemmatization.
We calculate the weights of words in the query using only term frequency. Now we retrieve the **top 10** reviews from the posting lists of each word in the search query. 
If a review appears in the top-10 elements of every query word, calculate cosine similarity score. 

$$ {sim(q,r)} = \\vec{q} \\cdot \\vec{d} = \\sum_{t\ \\text{in both q and r}} w\_{t,q} \\times w\_{t,r}.$$

Where **'q'** is the search query and **'r'** is a review vector. If  a review doesn't appear in the top-k elements of some query words, use the weight in the 10th element as the upper-bound on weight in vector. Hence, we can calculate the upper-bound score for using the query word's actual and upper-bound weights with respect to vector, as follows. 

$$ \\overline{sim(q,r)} = \\sum_{t\\in T_1} w\_{t,q} \\times w\_{t,r}.$$

$$  + \sum_{t\\in T_2} w\_{t,q} \\times \\overline{w\_{t,r}}.$$


In the above equation, first part has query words whose top-k elements contain review. Second part includes query words whose top-k elements do not contain the review. The weight in the 10-th element of word's postings list is used here. 

$$ \\overline{sim(q,r)} = \\sum_{t\\in q} w\_{t,q} \\times \\overline{w\_{t,r}}.$$

* We choose k as 20, this acts as a hyperparameter forthe search. Set k to be 20 as trial and error shows that this is optimal value.

* Let query be "My stomach hurts", After preprocessing the the vector representation will be ['stomach','hurt']

* Below we show how to algorithm calculate similarity of query.
<pre>
  word-weight-query = number of times word appears in the query / number of words in query


  1. retrive top 20 of posting list for each word in query
  2. find the list of all reviews. 
  3. for every review:
  4.   for every word:
  5.     if word exsists in the review:
  6.        score += word-weight-review * word-weight-query 
  7.     else:
  8.        score += word-weight-review20 * word-weight-query

</pre>

<h4> Contributions </h4>

1. Applied **nltk's WordNetLemmatizer**

2. Applied **nltk's stopwords** to reduce terms (In the reference there was manual stopword list which had very few words)

3. Applied cosine similarity.


<h2> Challenges faced </h2>

1. My data set had two version one was large(12GB), and another one was small (~300MB). But the smaller one didn't have the description of the products. While hosting in **pythonAnywhere** it only provides 512MB storage, while larger data set was **12GB** in size. To overcome this challenge, I parsed the large data set's json files for each product to extract the description and merged with smaller data set.

2. When the code was hosted in **pythonAnywhere**, there was no starting entry point for the client request. I have precalculated the **index** file(Containing TF, IDF, Postings for all terms and documents). After that I was loading the **index** file in memory, so that when a new query request comes, the results can be returned quickly. But as there was no starting entry point for initial code to run, there was no way to load the **index** _Ahead-of-Time_. 
I used caching this to load the **index** whenever a client request is performed first time after the backend is live. So during every next request, the **index** will remain loaded in memory.


<h3>Inverted Index vs Term Document Matrix</h3>
The term document matrix was generated with gensim library
* The retrive time for a search result was abot 10 seconds.
The custom Inverted Index yields results dractically faster
* The query retrive time was about 0.05 seconds


<h5> Referrences</h5>

*  http://www.ardendertat.com/2011/05/30/how-to-implement-a-search-engine-part-1-create-index/
*  https://stackabuse.com/python-for-nlp-creating-tf-idf-model-from-scratch/

</b>

