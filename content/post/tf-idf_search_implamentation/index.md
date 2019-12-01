
+++
title = "Implementing search engine using TF-IDF"
summary = "How I implemented a simple search engine using TF-IDF to search Lifestyle products"
+++
 
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
 
Searching for something is an inevitable part now. So, let's see, the basics of implementing a search engine. Here we will search for Life Style products which will based on [this](https://www.kaggle.com/paramaggarwal/fashion-product-images-small) data set which contains information on thousands of life style products. We will see _How to create **Index** and **rank** the results we get from the search_

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

### Inverted Index
An inverted index is a data structure that we build while parsing the documents that we are going to answer the search queries on. Given a query, we use the index to return the list of documents relevant for this query. The inverted index contains mappings from terms (words) to the documents that those terms appear in. Each vocabulary term is a key in the index whose value is its postings list. A term’s postings list is the list of documents that the term appears in.


### TF-IDF
Tf-idf is a weighting scheme that assigns each term in a document a weight based on its term frequency (tf) and inverse document frequency (idf).  The terms with higher weight scores are considered to be more important. It’s one of the most popular weighting schemes in Information Retrieval.


### Term Frequency – tf
Term Frequency is calculated for a term t in document d. It is basically the number of occurrences of the term in the document.

$$ tf\_{t,d} = N\_{t,d} $$

A term appears more in the document it becomes more important, which is logical. However, there is a drawback, by using term frequencies we lose positional information. The ordering of terms doesn’t matter, instead the number of occurrences becomes important. This is known as the bag of words model.

While using term frequencies if we use pure occurrence counts, longer documents will be favored more. So, we length normalize term frequencies. So, the term frequency of a term t in document D now becomes:

$$ tf\_{t,d} = \dfrac{N\_{t,d}}{||D||} $$


### Inverse Document Frequency – idf
We can’t only use term frequencies to calculate the weight of a term in the document, because tf considers all terms equally important. However, some terms occur more rarely and they are more discriminative than others. The idf of a term is the number of documents in the corpus divided by the document frequency of a term.

$$ idf_t = 1 + log\dfrac{N}{df_t} $$

### Tf-idf scoring
We have defined both tf and idf, and now we can combine these to produce the ultimate score of a term t in document d. We represent the document as a vector, with each entry being the tf-idf weight of the corresponding term in the document. The tf-idf weight of a term t in document d is simply the multiplication of its tf by its idf:

$$ tf\mbox{-}idf\_{t,d} = tf\_{t,d} \cdot idf_t $$


## Handling Query
If we compute the cosine similarity between the query vector and all the document vectors, sort them in descending order, and select the documents with top similarity, we will obtain an ordered list of relevant documents to this query.


To Summarize these steps in sudo code:

  Term frequency calculation:
  <pre>
  Normalization = sum(number of terms in doc)^2)
  term-tf-doc = sqrtnumber of times term appears in doc / sqrt(Normalization)
  
  </pre>
  Inverse document frequency for a term calculation:
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
### Phrase Query
The matching documents for phrase query are the ones that contain all query terms in the specified order. We will use the positional information about the terms. 
First we need the documents that all query terms appear in. We will again get the list of documents for each query term, but now we will take the intersection of them instead of union. 
Then, we should check whether they are in correct order or not. For each document that contains all query terms, we will do the following operations. 
 - Get positions of the query terms in the current document, and put each to a separate list. So, if there are n query terms, there will be n lists where each list contains the positions of the corresponding query term in the current document. 
 - Leave the position list of the 1st query term as it is, subtract 1 from each element of the 2nd position list, subtract 2 from each element of the 3rd position list, …, subtract n-1 from each element of nth position list. Then intersect all the position lists.
 - If the result of the intersection is non-empty, then all query terms appear in the current document in correct order, meaning this is a matching document.
 - Perform these operations on all documents that every query term appears in.
 
 The matching documents are the ones that have non-empty intersection.


### Highlighting the Query terms in result
To highlight the query words in the result, following steps were done.

- Split all terms in the doc
- For each doc terms, applied reduction and checked if it is available in the query term list
- If available replaced the **non reduced doc term** with  **<mark>non reduced doc term</mark>**
- Used a dictionary to avoid replacing multiple time for same doc term

## Contributions

1. Applied **nltk's WordNetLemmatizer**

2. Applied **nltk's stopwords** to reduce terms (In the reference there was manual stopword list which had around 70 less words)

3. Applied query term Highlighting

## Challenges faced

1. My data set had two version one was large(12GB), and another one was small (~300MB). But the smaller one didn't have the description of the products. While hosting in **pythonAnywhere** it only provides 512MB storage, while larger data set was **12GB** in size. To overcome this challenge, I parsed the large data set's json files for each product to extract the description and merged with smaller data set.

2. When the code was hosted in **pythonAnywhere**, there was no starting entry point for the client request. I have precalculated the **index** file(Containing TF, IDF, Postings for all terms and documents). After that I was loading the **index** file in memory, so that when a new query request comes, the results can be returned quickly. But as there was no starting entry point for initial code to run, there was no way to load the **index** _Ahead-of-Time_. 
I used caching this to load the **index** whenever a client request is performed first time after the backend is live. So during every next request, the **index** will remain loaded in memory.


## Stemming and Lemmatization (With vs Without)
* Tried different combination of word reduction. Comparisons are listed below:
<table style="width:100%">
  <tr>
    <th>Type of reduction</th>
    <th>Wordcount after reduction</th>
  </tr>
  <tr>
      <td>No reduction</td>
      <td align = "center">9580</td>
  </tr>
  <tr>
      <td>Only NLTK's stopwords removal</td>
      <td align = "center">9444</td>
  </tr>
  <tr>
      <td>Stopwords + Wordnet Lemmatizer</td>
      <td align = "center">8515</td>
  </tr>
  <tr>
      <td>Stopwords + Porter Stemming</td>
      <td align = "center">7097</td>
  </tr>
  <tr>
      <td>Stopwords + Porter Stemming + Lemmatization</td>
      <td align = "center">7080</td>
  </tr>
  <tr>
      <td>Stopwords + Snowball Stemming</td>
      <td align = "center">7057</td>
  </tr>
</table>


* Without lemmatization and stemming my search engine was not returning any result for queries that doesn't exactly match the data sets. Like for '_Narrowed_', there was no result, although there was products with '_narrow_' term in the description.

* Also without these pre-processing, there were around 25% more terms in the index, which increased the **index** file size by **50%**!! (6 MB without reduction, and ~4MB after reduction)


#### Referrences

*  http://www.ardendertat.com/2011/05/30/how-to-implement-a-search-engine-part-1-create-index/
*  https://stackabuse.com/python-for-nlp-creating-tf-idf-model-from-scratch/
*  https://www.youtube.com/watch?v=M-QRwEEZ9-8&t=226s
*  https://www.youtube.com/watch?v=IIi6e5oDZ68&t=439s





#### **[Source codes](https://github.com/anur0n/FashionStoreSearch)**

#### Read More
*  [Naive Bayes Text classifier](https://ashaduzzaman-rubel.netlify.com/post/naive-bayes-classifier/)
