
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

A term appears more in the document it becomes more important, which is logical. However, there is a drawback, by using term frequencies we lose positional information. The ordering of terms doesn’t matter, instead the number of occurrences becomes important. This is known as the bag of words model.

While using term frequencies if we use pure occurrence counts, longer documents will be favored more. So, we length normalize term frequencies. So, the term frequency of a term t in document D now becomes:

$$ {tf_t}_d = \dfrac{{N_t}_d}{||D||} $$


### Inverse Document Frequency – idf
We can’t only use term frequencies to calculate the weight of a term in the document, because tf considers all terms equally important. However, some terms occur more rarely and they are more discriminative than others. The idf of a term is the number of documents in the corpus divided by the document frequency of a term.

$$ idf_t = 1 + log\dfrac{N}{df_t} $$

### Tf-idf scoring
We have defined both tf and idf, and now we can combine these to produce the ultimate score of a term t in document d. We represent the document as a vector, with each entry being the tf-idf weight of the corresponding term in the document. The tf-idf weight of a term t in document d is simply the multiplication of its tf by its idf:

$$ tf\mbox{-}id{f_t}_d = {tf_t}_d \cdot idf_t $$

### Cosine Similarity
The similarity score between two vectors in a vector space is the the angle between them. If two documents are similar they will be close to each other in the vector space, having a small angle in between. This is called **Cosine Similarity**. To calculate cosine similarity We take the dot product of the vectors and the result is the cosine value of the angle between them. If we compute the cosine similarity between the query vector and all the document vectors, sort them in descending order, and select the documents with top similarity, we will obtain an ordered list of relevant documents to this query.


To Summarize in these steps in sudo code:

  Term frequency calculation:
  <pre>
  term-tf-doc = number of times term appears in doc / ||number of words in doc||
  </pre>
  Inverse document frequency for a word calculation:
  <pre>
  term-idf = 1 + log(total number of docs in the dataset / number of docs term occurs in)
  </pre>
  TF-IDF calculation:
  <pre>
  term-doc-score = term-tf-doc * term-idf
  </pre>


Following is the code used to prepare the index with TF-IDF


```
def createTfIdfIndex(self):
    self.prepareParams()

    with open(STYLE_WITH_DESC_N_TITLE, 'r', encoding='latin-1') as csvfile:
      reader = csv.reader(csvfile)

      for rowNo, row in enumerate((reader)):
        docId = row[COL_INDEX_ID]
        terms = self.getTerms(row[COL_INDEX_DESC_TITLE])
        if terms == None:
          continue

        self.titleIndex[docId] = row[COL_INDEX_DISPLAY_NAME]
        self.numberOfDocs += 1

        termDict = {}
        for position, term in enumerate(terms):
          try:
            termDict[term][1].append(position)
          except:
            termDict[term]=[docId, array('I',[position])]

        # Normalize the doc
        norm = 0
        for term, posting in termDict.items():
          norm += len(posting)**2
          norm = math.sqrt(norm)

        #calculate tf and df weights
        for term, posting in termDict.items():
          self.tf[term].append('%.4f' % (len(posting[1])/norm))
          self.df[term] += 1

        for term, posting in termDict.items():
          self.index[term].append(posting)

      self.writeIndexToFile()
      
def writeIndexToFile(self):
    '''write the inverted index to the file'''
    file=open(self.indexFile, 'w')
    print(self.numberOfDocs, file = file)
    self.numberOfDocs = float(self.numberOfDocs)

    for term in self.index.keys():
        postingList=[]
        for posting in self.index[term]:
            docID=posting[0]
            positions=posting[1]
            postingList.append(':'.join([str(docID) ,','.join(map(str,positions))]))
        postingData = ';'.join(postingList)
        tfData = ','.join(map(str, self.tf[term]))
        idfData = '%4f'%(1+np.log(self.numberOfDocs / self.df[term]))

        print('|'.join((term, postingData, tfData, idfData)), end="\n", file = file)
    file.close()
```






-----------------------------------------------

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

